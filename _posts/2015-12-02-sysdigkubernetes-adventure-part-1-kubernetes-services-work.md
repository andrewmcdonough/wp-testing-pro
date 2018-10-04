---
ID: 2297
post_title: 'A Sysdig + Kubernetes adventure, Part 1: How Kubernetes services work'
author: Gianluca Borello
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdigkubernetes-adventure-part-1-kubernetes-services-work/
published: true
post_date: 2015-12-02 09:00:50
---
We <a href="https://sysdigrp2rs.wpengine.com/digging-into-kubernetes-with-sysdig/" target="_blank">recently released</a> Kubernetes support for our open source visibility and troubleshooting tool, <a href="http://www.sysdig.org/" target="_blank">sysdig</a>, and the feedback from the community has been overwhelmingly positive - so today I want to continue the technical discussion on Kubernetes by going through a few examples of advanced Kubernetes troubleshooting. In particular, I’m going to focus on Kubernetes *services*. 

To refresh your memory, a service is a Kubernetes abstraction that works by providing a convenient and single entry point to access a group of Kubernetes pods. In other words, a service can be thought of as a dynamic load balancer for a set of pods (and the containers living inside them), automatically managed by the framework itself. 

I absolutely love services because they represent a very clean and consistent pattern that allows developers to write applications more efficiently, without having to worry about hand-crafted and bug-prone service discovery mechanisms. I am also surprised by how well they work - so much so that I decided to study them with sysdig! 

In this two-part series, we’ll first explore how services work from a technical point of view, to see how all the magic is implemented under the hood. In the second part, we’ll do a similar kind of exploration, but in faulty environments, trying to find the cause of various issues. 

## Setting up a Kubernetes service

I am a firm believer that in order to be effective at troubleshooting when anomalous conditions happen, one needs to be well aware of how the application works in a healthy environment: this way, by understanding the normal behavior of the system, spotting differences is much easier. Let’s start by studying how a service behaves in a fully working environment. 

I’m going to make the setup as simple as possible: it consists of two nginx pods, serving the standard index.html page that comes with nginx, and a single service that automatically balances the incoming traffic between them. 

Creating the pods is as simple as running: 

<pre>gianluca@sid:~$ kubectl run my-nginx --image=nginx --replicas=2 --port=80
CONTROLLER   CONTAINER(S)   IMAGE(S)   SELECTOR       REPLICAS
my-nginx     my-nginx       nginx      run=my-nginx   2
</pre>

And for the service: 

<pre>gianluca@sid:~$ kubectl expose rc my-nginx --port=8080 --target-port=80
NAME       LABELS         SELECTOR       IP(S)     PORT(S)
my-nginx   run=my-nginx   run=my-nginx             8080/TCP
</pre>

This last command creates a service, named my-nginx, that exposes a virtual IP address through which other pods in the cluster are transparently able to access the two backend nginx pods, using port 8080. This IP never changes for the entire lifespan of the service, which makes it incredibly useful for service discovery, because even if all my pods die and are rescheduled somewhere else, the entry point never changes and will not break the other parts of the application. I can get the virtual IP address with: 

<pre>gianluca@sid:~$ kubectl get services
NAME         LABELS                                    SELECTOR       IP(S)       PORT(S)
my-nginx     run=my-nginx                              run=my-nginx   10.0.0.142   8080/TCP
</pre>

But, since IP addresses are notably hard to remember, I prefer to use nice DNS names for my services, and Kubernetes has a very cool DNS extension that, once installed, runs a local DNS server inside the cluster and automatically translates DNS requests with the service name in it to the proper virtual IP address. Absolutely magic! 

Now that the infrastructure is up and running, before jumping into the low level details let’s run a pod with a simple Ubuntu container that doesn’t run anything other than a shell. I am going use this pod as a main access point to interact with my application from inside the cluster: 

<pre>gianluca@sid:~$ kubectl run -i --tty client --image=tutum/curl
Waiting for pod to be scheduled
Waiting for pod default/client-wd32o to be running, status is Pending, pod ready: false
Waiting for pod default/client-wd32o to be running, status is Running, pod ready: false
root@ubuntu:/#
</pre>

And once I’m in the shell I can finally interact with my service, using the DNS extension: 

<pre>root@ubuntu:/# curl my-nginx.default.cluster.local:8080
&lt; !DOCTYPE html>
&lt;html&gt;
&lt;head&gt;
&lt;title&gt;Welcome to nginx!&lt;/title&gt;
&lt;style&gt;
...&lt;/style&gt;
</pre>

## Digging into services

Now that the environment is set up, I’m going to take a sysdig capture while curl runs: 

<pre>gianluca@sid:~$ sudo sysdig -w trace.scap
</pre>

Once the capture has been taken, I am going to use sysdig to read it, visualizing only the network-related events: 

<pre>gianluca@sid:~$ sysdig -r trace.scap -A fd.type=ipv4 or fd.type=ipv6
</pre>

To start, I can see curl connecting to the DNS server, because obviously it needs to resolve the IP address for my-nginx.default.cluster.local: 

<pre>43340 15:40:58.352366394 3 curl (51152) > connect fd=3(&lt;4>)
43341 15:40:58.352375784 3 curl (51152) &lt; connect res=0 tuple=172.17.0.25:49950->172.16.189.136:53
</pre>

The DNS server that serves this request is, as predicted, the skydns process, which is part of the Kubernetes DNS extension and runs on my local machine (IP 172.16.189.136). We can see the binary data that is sent by curl, corresponding to a typical DNS request: 

<pre>43373 15:40:58.352568546 2 skydns (40700) > recvmsg fd=6(&lt;3t>:::53)
43374 15:40:58.352570915 2 skydns (40700) &lt; recvmsg res=48 size=48 data=
	0x0000: 0942 0100 0001 0000 0000 0000 086d 792d  .B...........my-
	0x0010: 6e67 696e 7807 6465 6661 756c 7407 636c  nginx.default.cl
	0x0020: 7573 7465 7205 6c6f 6361 6c00 001c 0001  uster.local.....
 tuple=::ffff:172.17.0.25:49950->::ffff:172.17.0.25:domain
 </pre>

And this is **where the magic starts happening**: the skydns process converts the DNS request to an HTTP request to etcd (port 4001) for the key “skydns/local/cluster/default/my-nginx”, to which etcd replies with the key value (in JSON), containing, among other things, the IP address of the service, 10.0.0.142, which is then returned to curl under the form of a DNS response: 

<pre>43405 15:40:58.353069087 2 skydns (40714) > write fd=3(&lt;4t>127.0.0.1:37238->127.0.0.1:4001) size=182
43406 15:40:58.353099313 2 skydns (40714) &lt; write res=182 data=
GET /v2/keys/skydns/local/cluster/default/my-nginx?quorum=false&recursive=true&sorted=false HTTP/1.1
Host: 127.0.0.1:4001
User-Agent: Go 1.1 package http
Accept-Encoding: gzip

43468 15:40:58.353384452 1 skydns (48700) > read fd=3(&lt;4t>127.0.0.1:37238->127.0.0.1:4001) size=4096
43469 15:40:58.353388313 1 skydns (48700) &lt; read res=406 data=
HTTP/1.1 200 OK
Content-Type: application/json
X-Etcd-Cluster-Id: f68652439e3f8f2a
X-Etcd-Index: 166
X-Raft-Index: 1205
X-Raft-Term: 2
Date: Thu, 05 Nov 2015 23:40:58 GMT
Content-Length: 205

{"action":"get","node":{"key":"/skydns/local/cluster/default/my-nginx","value":"{\"host\":\"10.0.0.142\",\"priority\":10,\"weight\":10,\"ttl\":30,\"targetstrip\":0}","modifiedIndex":70,"createdIndex":70}}
</pre>

Now that curl got the IP address of the service from skydns, it can connect to it on port 8080 and send the HTTP request: </pre>

<pre>43645 15:40:58.356720087 2 curl (51151) &lt; connect res=-115(EINPROGRESS) tuple=172.17.0.25:50950->10.0.0.142:http-alt
43763 15:40:58.357571155 2 curl (51151) > sendto fd=3(&lt;4t>172.17.0.25:50950->10.0.0.142:8080) size=99 tuple=NULL
43772 15:40:58.357612691 2 curl (51151) &lt; sendto res=99 data=
GET / HTTP/1.1
User-Agent: curl/7.35.0
Host: my-nginx.default.cluster.local:8080
Accept: */*
</pre>

However, this is where things get a bit odd. As you can see next, the hyperkube process seems to intercept the connection that curl sent, and read the original HTTP request from it. In particular, notice how the source port seems to be in both cases 50950, but the destination IP address and port seem to have magically changed from 10.0.0.142:8080 to 172.17.42.1:39551: </pre>

<pre>43795 15:40:58.357730454 2 hyperkube (39579) > read fd=10(&lt;4t>172.17.0.25:50950->172.17.42.1:39551) size=32768
43796 15:40:58.357733201 2 hyperkube (39579) &lt; read res=99 data=
GET / HTTP/1.1
User-Agent: curl/7.35.0
Host: my-nginx.default.cluster.local:8080
Accept: */*
</pre>

The trick used by Kubernetes here is to leverage the Linux kernel Netfilter capabilities to create a virtual IP that never leaves the host. As you can read from my iptables configuration below, every time someone tries to connect to 10.0.0.142:8080, it will be redirected to 172.16.189.136:39551, where the Kubernetes proxy server (shown as hyperkube in my sysdig trace file) will get the request and will properly route it to one of the backend nginx pods: </pre>

<pre>gianluca@sid:~$ sudo iptables -t nat -L
...
Chain KUBE-PORTALS-CONTAINER (1 references)
target     prot opt source               destination
...
REDIRECT   tcp  --  anywhere             10.0.0.142           /* default/my-nginx: */ tcp dpt:http-alt redir ports <b>39551</b>
...

Chain KUBE-PORTALS-HOST (1 references)
target     prot opt source               destination
...
DNAT       tcp  --  anywhere             10.0.0.142           /* default/my-nginx: */ tcp dpt:http-alt to:172.16.189.136:<b>39551</b>
...
</pre>

Once the proxy is done reading the HTTP request, it can normally forward it to one of the nginx pods (172.17.0.24 in this case), where the nginx process can serve the transaction: 

<pre>43798 15:40:58.357735649 2 hyperkube (39579) > write fd=11(&lt;4t>172.17.42.1:38546->172.17.0.24:80) size=99
43799 15:40:58.357755452 2 hyperkube (39579) &lt; write res=99 data=
GET / HTTP/1.1
User-Agent: curl/7.35.0
Host: my-nginx.default.cluster.local:8080
Accept: */*

43907 15:40:58.358733815 2 nginx (40473) > recvfrom fd=3(&lt;4t>172.17.42.1:38546->172.17.0.24:80) size=1024
43908 15:40:58.358735877 2 nginx (40473) &lt; recvfrom res=99 data=
GET / HTTP/1.1
User-Agent: curl/7.35.0
Host: my-nginx.default.cluster.local:8080
Accept: */*
</pre>

And this is how Kubernetes services are implemented! Notice that there might be different behaviors depending on the specific Kubernetes configuration and version, but this represents a very common behavior. 

## Conclusions

Today we explored a pretty advanced use case, and we went a long way understanding one of the trickiest parts of Kubernetes, all with sysdig. If we pair this with the Kubernetes support described in <a href="https://sysdigrp2rs.wpengine.com/digging-into-kubernetes-with-sysdig/" target="_blank">our previous blog post</a>, we can immediately see how sysdig can be helpful in a variety of situations, from basic Kubernetes exploration with csysdig’s views to advanced process troubleshooting with sysdig’s system and Kubernetes filters. And if you want to keep an eye on your whole multi-node Kubernetes cluster, we even have Sysdig Cloud! 

In part two, we’ll leverage all the knowledge gained here to troubleshoot faulty environments where Kubernetes services are misbehaving. Stay tuned! </pre>