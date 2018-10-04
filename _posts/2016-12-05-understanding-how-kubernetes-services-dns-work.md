---
ID: 3405
post_title: >
  Understanding how Kubernetes DNS
  services work
author: Jorge Salamero Sanz
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/understanding-how-kubernetes-services-dns-work/
published: true
post_date: 2016-12-05 08:12:03
---
Kubernetes allows you to create container groups and define <a href="http://kubernetes.io/docs/user-guide/services/" title="Kubernetes Services" target="_blank">services</a> on top of them. Kubernetes assigns each service a virtual static IP address routable within the cluster, so any connection that reaches this IP address will be automatically routed to one of the containers in the group.

The benefit of using services is that you are able to access the functionality provided by the containers without knowing their identity. This is basically an easy to discover load balancer. And to make things even easier, Kubernetes also generates an internal DNS entry that resolves to this IP address.

When it’s all in place it feels a little bit like magic. But, looking under the hood, it’s easy to understand what Kubernetes is doing. With a little more work - and digging into system calls - we can see how Kubernetes goes about it. Let’s start with the what and then go into the how.

## Deploying a Simple Service

We are going to deploy a simple service called backend (you can <a href="https://github.com/gianlucaborello/healthchecker-kubernetes/blob/master/manifests/backend.yaml" target="_blank">download it from here</a>) within a [namespace][1] critical-app. The backend service is the entry point to 3 Nginx pods grouped in what Kubernetes calls a <a href="http://kubernetes.io/docs/user-guide/deployments/" title="Kubernetes Deployments" target="_blank">deployment</a>.

    $ kubectl create namespace critical-app
    namespace "critical-app" created
    $ kubectl create -f backend.yaml
    service "backend" created
    deployment "backend" created
    

Now we can use the describe command to show the IP address assigned to the service:

    $ kubectl describe service backend --namespace critical-app
    Name:			backend
    Namespace:		critical-app
    Labels:		        app=critical-app
    			role=backend
    Selector:		app=critical-app,role=backend
    Type:			ClusterIP
    IP:			10.3.0.116
    Port:			<unset>	80/TCP
    Endpoints:		172.17.0.4:80,172.17.0.5:80,172.17.0.6:80
    Session Affinity:	None
    

Here we can also identify the 3 underlying pods IP addresses as the Endpoints.

Since we wanted to see the Kubernetes service magic in action, let’s create an interactive container with curl installed to ping the backend entry point:

    $ kubectl run -it --image=tutum/curl client --namespace critical-app --restart=Never
    

This will drop me in an interactive shell. What if I try to access my backend through its name?

    root@client:/# curl backend
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    [...]
    </html>

Automagically I get a response from one of the underlying Nginx containers. I didn't have to know the identity, Kubernetes DNS did everything for me.

## How do Kubernetes services work?

In order to better understand what’s happening behind the scenes, let’s take a look at the underlying machinery that creates that magical effect.

In order to do this we will use Sysdig, the open source container troubleshooting tool, to see Kubernetes in action from the perspective of underlying system calls.

Sysdig allows us to troubleshoot processes more effectively by correlating container-aware system events with rich metadata coming from the Kubernetes API server (pods, services, deployments, etc). Think of all your favourite system troubleshooting tools: strace, htop, lsof, netstat, vmstat, iostat, tcpdump… all working together and understanding your containers.

<a href="http://www.sysdig.org/install/" title="How to install Sysdig" taget="_blank">Installing Sysdig</a> is really easy. Sysdig can run either from the host or you can use Sysdig Docker image running as a privileged container. If you have never used sysdig with Kubernetes before, it’s probably a very good idea going first through <a href="https://sysdigrp2rs.wpengine.com/blog/digging-into-kubernetes-with-sysdig/" title="Digging into Kubernetes with Sysdig" target="_blank">Digging into Kubernetes with Sysdig</a>.

Assuming you know the basic already, let’s run sysdig on our Kubernetes host:

    $ sudo sysdig -k http://localhost:8080 -pk -s8192 -NA \
    "fd.type in (ipv4, ipv6) and (k8s.ns.name=critical-app or proc.name=skydns)"

I’ll explain each parameter here: -k http://localhost:8080 connects to Kubernetes API -pk prints Kubernetes fields in stdout -s8192 enlarges the IO buffers, as we need to show full content, otherwise gets cut off by default -NA shows ASCII output And the filter between double quotes. Sysdig is able to understand Kubernetes semantics so we can filter out traffic on sockets IPv4 or IPv6, coming from any container in the namespace critical-app or from any process named skydns. We included proc.name=skydns because this is the internal Kubernetes DNS resolver and runs outside our namespace, as part of the Kubernetes infrastructure.

Now, let's run curl again with sysdig running in parallel and we'll see some interesting information on sysdig output.

    30447 11:45:26.135742024 0 client (341cbd92c83b) curl (29549:14) < socket fd=3(<6>)
    30448 11:45:26.135744012 0 client (341cbd92c83b) curl (29549:14) > close fd=3(<6>)
    30449 11:45:26.135746067 0 client (341cbd92c83b) curl (29549:14) < close res=0
    30615 11:45:26.136987377 1 client (341cbd92c83b) curl (29550:15) < socket fd=3(<4>)
    30616 11:45:26.136994479 1 client (341cbd92c83b) curl (29550:15) > connect fd=3(<4>)
    30617 11:45:26.137006538 1 client (341cbd92c83b) curl (29550:15) < connect res=0 tuple=172.17.0.7:49325->10.0.2.15:53
    30628 11:45:26.137187035 1 <NA> (659d266db640) skydns (12139:1) > recvmsg fd=6(<3t>:::53)
    30636 11:45:26.137227118 1 <NA> (659d266db640) skydns (12139:1) < recvmsg res=56 size=56 data=
    backendcritical-appsvcclusterlocal tuple=::ffff:172.17.0.7:49325->:::53

In this output we can identify how the curl process in the container client generates a DNS request which is received by a process named skydns, a DNS server listening on port 53. Looking at the payload we can see the DNS query was for backend.critical-app.svc.cluster.local. curl understood that backend wasn't a fully qualified name and it appended .critical-app.svc.cluster.local so Kubernetes could actually resolve it.

    30640 11:45:26.137499776 1 <NA> (659d266db640) skydns (12139:1) > write fd=3(<4t>10.0.2.15:40764->10.0.2.15:4001) size=190
    30641 11:45:26.137543936 1 <NA> (659d266db640) skydns (12139:1) < write res=190 data=
    GET /v2/keys/skydns/local/cluster/svc/critical-app/backend?quorum=false&recursive=true&sorted=false HTTP/1.1
    Host: 10.0.2.15:4001
    User-Agent: Go 1.1 package http
    Accept-Encoding: gzip
    
     
    30642 11:45:26.137555394 1 <NA> (659d266db640) skydns (12139:1) > recvmsg fd=6(<3t>:::53)
    30685 11:45:26.138173278 1 <NA> (659d266db640) skydns (12154:8) > read fd=3(<4t>10.0.2.15:40764->10.0.2.15:4001) size=4096
    30686 11:45:26.138179138 1 <NA> (659d266db640) skydns (12154:8) < read res=553 data=
    HTTP/1.1 200 OK
    Content-Type: application/json
    X-Etcd-Cluster-Id: 7e27652122e8b2ae
    X-Etcd-Index: 15081
    X-Raft-Index: 42991
    X-Raft-Term: 2
    Date: Fri, 02 Dec 2016 11:45:26 GMT
    Content-Length: 349
    
    {"action":"get","node":{"key":"/skydns/local/cluster/svc/critical-app/backend","dir":true,"nodes":[{"key":"/skydns/local/cluster/svc/critical-app/backend/5c82f78b","value":"{\"host\":\"10.3.0.116\",\"priority\":10,\"weight\":10,\"ttl\":30,\"targetstrip\":0}","modifiedIndex":14027,"createdIndex":14027}],"modifiedIndex":14027,"createdIndex":14027}}
    [...]
    
    

The ball was in SkyDNS court because had to resolve this DNS request to an IP address. How does it do that? In Kubernetes, the source of truth is stored in etcd, a distributed key-value datastore that contains all the settings.

We can see how SkyDNS starts doing a connection towards etcd (port 4001). We cannot go further because it is beyond our filtering scope (remember we only include our namespace and from the outside world only processes named skydns). But we can see how it makes a HTTP request to the etcd API asking for the key at /local/cluster/svc/critical-app/backend. etcd replies with a JSON that includes the IP address of the backend service. Then, SkyDNS makes a DNS response with the IP address, makes it way back to the curl client container.

    30733 11:45:26.140184272 0 client (341cbd92c83b) curl (29549:14) > connect fd=3(<4>)
    30738 11:45:26.140535260 0 backend-1440326531-o05qw (4f498dee9e9c) nginx (20647:7) < accept fd=3(<4t>172.17.0.7:46306->172.17.0.6:80) tuple=172.17.0.7:46306->172.17.0.6:80 queuepct=0 queuelen=0 queuemax=128
    30743 11:45:26.140586592 0 client (341cbd92c83b) curl (29549:14) < connect res=-115(EINPROGRESS) tuple=172.17.0.7:46306->10.3.0.116:80
    30752 11:45:26.140626292 0 client (341cbd92c83b) curl (29549:14) > sendto fd=3(<4t>172.17.0.7:46306->10.3.0.116:80) size=71 tuple=NULL
    30753 11:45:26.140674832 0 client (341cbd92c83b) curl (29549:14) < sendto res=71 data=
    GET / HTTP/1.1
    User-Agent: curl/7.35.0
    Host: backend
    Accept: */*
    [...]

Finally curl does a connect to the resolved IP address and the HTTP request comes through. If you want to see the full trace, it's <a href="https://gist.github.com/bencer/a6dfdf30ea991ca5ca610f2e090ec48a" target="_blank">available here</a>.

What is interesting here is that we can see how nginx in one of my underlying containers accepts the HTTP request. But if you pay closer attention, you will notice how the IP addresses are different: curl opened a connection towards 10.3.0.116:80 but nginx IP address is 172.17.0.6:80. The connection has been hijacked by Kubernetes implementing the load balancing with iptables. If we check the NAT table:

    $ sudo iptables -n -t nat -L 
    Chain PREROUTING (policy ACCEPT)
    target     prot opt source               destination         
    KUBE-SERVICES  all  --  0.0.0.0/0            0.0.0.0/0            /* kubernetes service portals */
    DOCKER     all  --  0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL
    
    [...]
    
    Chain KUBE-SERVICES (2 references)
    target     prot opt source               destination         
    [...]
    KUBE-SVC-QIAMZM7CZ7DGYY4U  tcp  --  *      *       0.0.0.0/0            10.3.0.116           /* critical-app/backend: cluster IP */ tcp dpt:80
    
    [...]
    
    Chain KUBE-SVC-QIAMZM7CZ7DGYY4U (1 references)
    target     prot opt source               destination         
    KUBE-SEP-3EH5NW6TCZ37SLM7  all  --  0.0.0.0/0            0.0.0.0/0            /* critical-app/backend: */ statistic mode random probability 0.33332999982
    KUBE-SEP-TM67DFGZSWKXXTT4  all  --  0.0.0.0/0            0.0.0.0/0            /* critical-app/backend: */ statistic mode random probability 0.50000000000
    KUBE-SEP-4M5CAF23423KHRN7  all  --  0.0.0.0/0            0.0.0.0/0            /* critical-app/backend: */
    
    [...]
    
    Chain KUBE-SEP-3EH5NW6TCZ37SLM7 (1 references)
    target     prot opt source               destination         
    KUBE-MARK-MASQ  all  --  172.17.0.4           0.0.0.0/0            /* critical-app/backend: */
    DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            /* critical-app/backend: */ tcp to:172.17.0.4:80
    
    [...]
    
    Chain KUBE-SEP-4M5CAF23423KHRN7 (1 references)
    target     prot opt source               destination         
    KUBE-MARK-MASQ  all  --  172.17.0.6           0.0.0.0/0            /* critical-app/backend: */
    DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            /* critical-app/backend: */ tcp to:172.17.0.6:80
    
    Chain KUBE-SEP-TM67DFGZSWKXXTT4 (1 references)
    target     prot opt source               destination         
    KUBE-MARK-MASQ  all  --  172.17.0.5           0.0.0.0/0            /* critical-app/backend: */
    DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            /* critical-app/backend: */ tcp to:172.17.0.5:80

We can see how traffic to the service IP address is redirected into a chain that using iptables probabilistic module rewrites the traffic to the 3 pods IP addresses. We have just discovered how Kubernetes implements stateless load balancing :).

What follows it is just the replies coming back, Nginx writes to the socket and curl will receive the HTTP response.

## Conclusion

We have learned how SkyDNS resolves DNS requests asking etcd HTTP API and how Kubernetes implements stateless load balancing using iptables. More broadly, we have learned not only how to spy traffic on network sockets but in a container-aware fashion.

In fact, in part 2, we’ll use this capability to actually troubleshoot a problem with Kubernetes DNS resolution. Subscribe to our blog to be informed when the next post is available.

 [1]: http://kubernetes.io/docs/user-guide/namespaces/ "Kubernetes Namespaces"