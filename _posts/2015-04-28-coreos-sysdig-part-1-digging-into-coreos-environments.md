---
ID: 1556
post_title: 'CoreOS &#038; Sysdig, Part 1: Digging Into CoreOS Environments'
author: Gianluca Borello
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/coreos-sysdig-part-1-digging-into-coreos-environments/
published: true
post_date: 2015-04-28 06:00:43
---
We recently released container support in sysdig, and the reaction from the community has been extremely positive. Many sysdig users have asked us about support for CoreOS. We love CoreOS! And, like many new and powerful technologies, CoreOS presents some unique challenges when it comes to visibility and troubleshooting. Sysdig is here to help :) 

In this two-part series, I want to introduce you to the unique visibility sysdig can offer into CoreOS environments. We’ll start by walking through an actual example of deploying sysdig in a representative CoreOS environment. If you’re curious about how to get sysdig working with CoreOS, then start here. As a proof of concept, we’ll explore two simple, but powerful use cases: 

*   Observing application transactions across containers
*   Observing commands run inside a container



Or you can jump ahead to the more advanced CoreOS troubleshooting scenarios in <a href="https://sysdigrp2rs.wpengine.com/?p=1557" target="_blank">part two of this series</a>, where we explore the inner workings of key CoreOS components like etcd, flannel, and confd. 

Note, for the raw docs on how to install sysdig with CoreOS: <a href="http://www.sysdig.org/install/" target="_blank">see here</a>. 

## Installing Sysdig On CoreOS 

The recommended way to run sysdig on CoreOS is inside of its own Docker container. 

<a href="https://sysdigrp2rs.wpengine.com/let-light-sysdig-adds-container-visibility/" target="_blank">Remember</a>, even if sysdig is running inside a container, **you will still be able to see all the system events for all the processes in the system**, even if they are running in the host or inside another container. You get full visibility into every other container, without having to pollute any other container with extra instrumentation. 

For one-step sysdig installation with CoreOS, we provide a specially designed Docker container here: <a href="https://registry.hub.docker.com/u/sysdig/sysdig/" title="https://registry.hub.docker.com/u/sysdig/sysdig/" target="_blank">https://registry.hub.docker.com/u/sysdig/sysdig/</a> 

Let’s run the installation: 

<pre>core@balancer ~ $ docker run -i -t --name sysdig --privileged -v /var/run/docker.sock:/host/var/run/docker.sock -v /dev:/host/dev -v /proc:/host/proc:ro sysdig/sysdig
...
Download succeeded, loading module
root@7a197036f700:/# sysdig
7 19:58:54.856516203 0 sysdig (3249) &gt; switch next=0 pgft_maj=0 pgft_min=984 vm_size=43264 vm_rss=5260 vm_swap=0
...
</pre>

Overall, this Docker install method is nice because it’s automatically updated by us*, it has some nice features such as automatic setup and bash completion, and it’s a generic approach that can be used on other distributions outside CoreOS as well. 

However, we are aware that some users might want to install sysdig in the CoreOS toolbox, and this can be easily achieved by normally installing sysdig inside the toolbox, and then manually running the *sysdig-probe-loader* script: 

<pre>$ toolbox --bind=/dev --bind=/var/run/docker.sock
# curl -s https://s3.amazonaws.com/download.draios.com/stable/install-sysdig | bash
# sysdig-probe-loader
...
Download succeeded, loading module
# sysdig
7 19:58:54.856516203 0 sysdig (3249) &gt; switch next=0 pgft_maj=0 pgft_min=984 vm_size=43264 vm_rss=5260 vm_swap=0
…
</pre>

## Setting up a basic CoreOS infrastructure

Now that we’ve got sysdig up and running on our CoreOS environment, I’m going to deploy a very simple infrastructure to demonstrate some use cases. This is the basic cloud-config file for CoreOS that I’m using for spinning up two hosts: 

<pre>#cloud-config

hostname: balancer
coreos:
  fleet:
    metadata: role=balancer
  etcd:
    discovery: https://discovery.etcd.io/863498934f8fec033740b93bdebac5a9
    addr: $private_ipv4:4001
    peer-addr: $private_ipv4:7001
  units:
    - name: flanneld.service
      drop-ins:
        - name: 50-network-config.conf
          content: |
            [Service]
            ExecStartPre=/usr/bin/etcdctl set /coreos.com/network/config '{ "Network": "192.168.0.0/16" }'
      command: start
    - name: etcd.service
      command: start
    - name: fleet.service
      command: start
</pre>

Here we see my “balancer” host and, by replacing the hostname and metadata, I can create another, called “frontend”. 

The architecture I’m going to deploy covers a few patterns that are common among CoreOS users. In particular, there are multiple frontend containers, serving static pages, that write their own IP addresses, upon start, into etcd, the CoreOS distributed key-value database. These containers run on the cluster node called “frontend”. There is then one haproxy container, balancing traffic between all the frontend containers. Haproxy is automatically configured via confd, a daemon that watches a hierarchy of etcd keys and, upon changes, regenerates on the fly the haproxy configuration and restarts it. 

These basic building blocks are at the core of modern containerized infrastructures based on micro services and automatic service discovery, so this is should be a good example to work on. 

To deploy the services, I’ll use Fleet unit files, such as “frontend@.service”, for the frontend containers: 

<pre>[Unit]
Description=frontend%i
After=docker.service

[Service]
TimeoutStartSec=0
ExecStartPre=-/usr/bin/docker kill frontend%i
ExecStartPre=-/usr/bin/docker rm -v frontend%i
ExecStartPre=/usr/bin/docker pull nginx
ExecStart=/usr/bin/docker run --name frontend%i nginx
ExecStartPost=/bin/sh -c "sleep 2 && etcdctl set /haproxy-discover/services/myapp/upstreams/frontend%i $(docker inspect --format '{{ .NetworkSettings.IPAddress }}' frontend%i):80"
ExecStartPost=/usr/bin/etcdctl set /haproxy-discover/services/myapp/domain myapp.org
ExecStop=/usr/bin/etcdctl rm /haproxy-discover/services/myapp/upstreams/frontend%i
ExecStop=/usr/bin/docker stop frontend%i

[X-Fleet]
MachineMetadata="role=frontend"
</pre>

...and "haproxy-confd.service", for the haproxy service: 

<pre>[Unit]
Description=haproxy-confd
After=docker.service

[Service]
TimeoutStartSec=0
ExecStartPre=/usr/bin/etcdctl mkdir /haproxy-discover/tcp-services
ExecStartPre=-/usr/bin/docker kill haproxy-confd
ExecStartPre=-/usr/bin/docker rm -v haproxy-confd
ExecStartPre=/usr/bin/docker pull yaronr/haproxy-confd
ExecStart=/usr/bin/docker run --net=host -e ETCD_NODE=127.0.0.1:4001 --name haproxy-confd yaronr/haproxy-confd
ExecStop=/usr/bin/docker stop haproxy-confd
ExecStop=/usr/bin/etcdctl rmdir /haproxy-discover/tcp-services

[X-Fleet]
MachineMetadata="role=balancer"
</pre>

Deploying the infrastructure, with four frontend units and the haproxy balancer, is as simple as running: 

<pre>fleetctl start haproxy-confd
fleetctl start frontend@{1..4}
</pre>

After deployment, these are the IP addresses of hosts and containers: 

*   balancer host: 10.1.1.95 (flannel subnet 192.168.69.0/24)

*   haproxy container: 10.1.1.95

*   frontend host: 10.1.1.187 (flannel subnet 192.168.13.0/24)
*   frontend1 container: 192.168.13.2
*   frontend2 container: 192.168.13.3
*   frontend3 container: 192.168.13.4
*   frontend4 container: 192.168.13.5

And that’s it: now we have sysdig and a full infrastructure running in CoreOS together! Let’s see just a taste of what sysdig is capable of now. 

## Observing application transactions across containers 

Sysdig can be used to fully inspect, for example, an HTTP transaction end-to-end, inside or outside containers, and this easily works on CoreOS as well. If I run: 

<pre>core@balancer ~ $ curl -H 'Host:myapp.org' localhost:8080
&lt;!DOCTYPE html&gt;<br />&lt;html&gt;<br />&lt;head&gt;<br />&lt;title&gt;Welcome to nginx!&lt;/title&gt;<br />…
</pre>

...then I can then use sysdig with the echo_fds chisel to see the entire path of the transaction: 

<pre># sysdig -pc -A -c echo_fds 'fd.type=ipv4 and (proc.name=curl or proc.name=haproxy)'
------ Write 72B to  [host] [host]  127.0.0.1:49100-&gt;127.0.0.1:8080 (curl)
GET / HTTP/1.1
User-Agent: curl/7.30.0
Accept: */*
Host:myapp.org
------ Read 72B from  [haproxy-confd] [3f9e28ee47db]  127.0.0.1:49100-&gt;127.0.0.1:8080 (haproxy)
GET / HTTP/1.1
User-Agent: curl/7.30.0
Accept: */*
Host:myapp.org
------ Write 100B to  [haproxy-confd] [3f9e28ee47db]  192.168.69.0:52279-&gt;192.168.13.2:80 (haproxy)
GET / HTTP/1.1
User-Agent: curl/7.30.0
Accept: */*
Host:myapp.org
X-Forwarded-For: 127.0.0.1
------ Read 238B from  [haproxy-confd] [3f9e28ee47db]  192.168.69.0:52279-&gt;192.168.13.2:80 (haproxy)
HTTP/1.1 200 OK
Server: nginx/1.7.10
Date: Wed, 25 Mar 2015 20:35:03 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 10 Feb 2015 15:19:43 GMT
Connection: keep-alive
ETag: \"54da218f-264\"
Accept-Ranges: bytes

&lt;!DOCTYPE html&gt;<br />&lt;html&gt;<br />&lt;head&gt;
...
------ Write 238B to  [haproxy-confd] [3f9e28ee47db]  127.0.0.1:49100-&gt;127.0.0.1:8080 (haproxy)
HTTP/1.1 200 OK
Server: nginx/1.7.10
Date: Wed, 25 Mar 2015 20:35:03 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 10 Feb 2015 15:19:43 GMT
Connection: keep-alive
ETag: \"54da218f-264\"
Accept-Ranges: bytes

&lt;!DOCTYPE html&gt;<br />&lt;html&gt;<br />&lt;head&gt;
...
------ Read 850B from  [host] [host]  127.0.0.1:49100-&gt;127.0.0.1:8080 (curl)
HTTP/1.1 200 OK
Server: nginx/1.7.10
Date: Wed, 25 Mar 2015 20:35:03 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 10 Feb 2015 15:19:43 GMT
Connection: keep-alive
ETag: \"54da218f-264\"
Accept-Ranges: bytes

&lt;!DOCTYPE html&gt;<br />&lt;html&gt;<br />&lt;head&gt;
...
</pre>

I can clearly see that curl contacts haproxy, which lives inside the haproxy-confd container (sysdig shows the container name and container id), and haproxy forwards the request to one of the frontend containers (frontend1) and waits for the HTTP response, which is then sent back to curl. Pretty cool, huh? ;) This is super helpful when troubleshooting containerized applications, especially in production since this method is completely passive. 

## Observing commands run inside a container 

Another interesting thing that can save a bunch of time when troubleshooting failing deployments is intercepting what is actually executed inside a container. For example, let’s try watching an actual unit deployment. When I schedule the haproxy-confd container to start with fleet, I can see what commands are executed and where: 

<pre># sysdig -pc -c spy_users
629 13:34:29 core@host) fleetctl start haproxy-confd.service
4280 13:34:34 root@haproxy-confd) confd -onetime -node 127.0.0.1:4001
        4280 13:34:34 root@haproxy-confd) /bin/sh -c /usr/sbin/haproxy -c -f /etc/haproxy/.haproxy.cfg495230897
            4280 13:34:34 root@haproxy-confd) /usr/sbin/haproxy -c -f /etc/haproxy/.haproxy.cfg495230897
        4280 13:34:34 root@haproxy-confd) /bin/sh -c haproxy -f /etc/haproxy/haproxy.cfg -p /var/run/haproxy.pid -D -sf $(cat /var/run/haproxy.pid)
            4280 13:34:34 root@haproxy-confd) cat /var/run/haproxy.pid
            4280 13:34:34 root@haproxy-confd) haproxy -f /etc/haproxy/haproxy.cfg -p /var/run/haproxy.pid -D -sf
4280 13:34:34 root@haproxy-confd) confd -node 127.0.0.1:4001
</pre>

In this case, I see that confd is first executed in onetime mode, and in daemon mode right after. Pretty useful especially for complicated entry points. 

## Conclusion 

Hopefully you now have a basic idea of how to get up and running with sysdig and CoreOS. With the addition of a single sysdig container, you can easily get unprecedented visibility into your whole CoreOS environment. Containerized systems are naturally quite opaque, and I promise you: what we covered today is really just the tip of the iceberg for sysdig + CoreOS. In part 2 of this series, I’ll cover more advanced troubleshooting scenarios involving etcd, flannel networking and confd. 

And finally, some of you may be wondering how you can get unified visibility across your distributed CoreOS infrastructure... check out <a href="https://sysdigrp2rs.wpengine.com/landing-quote2/?utm_source=web&utm_medium=blog&utm_campaign=COSBlog1-042815" target="_blank">Sysdig Cloud</a> ;) Sysdig Cloud is the first ever monitoring and alerting solution designed from the ground up to support containerized and CoreOS environments. 

Happy digging! 

_______________________ 

*As most sysdig users are aware, sysdig depends on a little kernel module, *sysdig-probe*, in order to work properly. Compiling *sysdig-probe* is usually handled by the Linux package manager, DKMS, but since CoreOS is so lightweight, DKMS has been stripped out. When this sysdig Docker container starts, it immediately runs a helper script, *sysdig-probe-loader*, which automatically downloads a precompiled version of *sysdig-probe* for your exact kernel (version and configuration) from our HTTPS repository. At this moment, we automatically maintain builds for all the CoreOS releases (our continuous integration servers periodically check for a new CoreOS release, and precompile and upload the sysdig-probe kernel module as needed).