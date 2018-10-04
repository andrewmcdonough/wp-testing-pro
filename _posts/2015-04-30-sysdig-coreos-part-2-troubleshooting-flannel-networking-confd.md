---
ID: 1557
post_title: 'Sysdig &#038; CoreOS, Part 2: Troubleshooting Flannel Networking &#038; Confd'
author: Gianluca Borello
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-coreos-part-2-troubleshooting-flannel-networking-confd/
published: true
post_date: 2015-04-30 06:00:11
---
Welcome to the second half of this two-part series exploring the unique visibility sysdig can offer into CoreOS environments. If you haven’t already done so, I encourage you to check out <a href="https://sysdigrp2rs.wpengine.com/?p=1556" target="_blank">part 1 of this series</a> which covered: 

*   Sysdig deployment in CoreOS
*   Setting up a basic CoreOS infrastructure
*   Observing application transactions across containers
*   Observing commands run inside a container

In this post, I’ll focus on the more advanced topics of troubleshooting flannel networking and confd. Hopefully, this will also offer an interesting, practical overview of how these services work. I will also demonstrate a few tricks to troubleshoot some of the CoreOS-specific issues and bugs that can bite you during your quest to create the coolest microservice-based architecture in the world. 

## Troubleshooting flannel networking 

In <a href="https://sysdigrp2rs.wpengine.com/?p=1556" target="_blank">part 1 of this series</a> we deployed a basic but representative CoreOS environment with the following IP addresses of hosts and containers: 

*   balancer host: 10.1.1.95 (flannel subnet 192.168.69.0/24)

*   haproxy container: 10.1.1.95

*   frontend host: 10.1.1.187 (flannel subnet 192.168.13.0/24)
*   frontend1 container: 192.168.13.2
*   frontend2 container: 192.168.13.3
*   frontend3 container: 192.168.13.4
*   frontend4 container: 192.168.13.5

Then we used sysdig (with the echo_fds chisel) to trace the path of an HTTP transaction through our environment: 

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

If you pay close attention to this transaction, there’s one piece of it that looks very odd: how can haproxy directly contact the frontend container with IP 192.168.13.2 (frontend1), which lives on another host? That shouldn’t be possible, since no one else from an external host should be able to connect to it. Let's see what's going on here. 

By default Docker containers get a private network for themselves, so all the containers inside a specific host can talk to each other using the private IP addresses from this network. However, this becomes challenging when containers from different hosts need to communicate, because the IP addresses of the containers won’t be reachable from other remote containers. Traditionally this problem is solved using port forwarding via iptables. This means that, in order to reach a remote container, one can just connect to a specific port on the IP address of the host running that container. But this is less than ideal and it’s usually considered a pain. 

However, the CoreOS hosts I deployed are running Flannel: Flannel creates a virtual network that spans across all the containers in the cluster, making direct remote communication among containers possible. In this case, the network that I configured via cloud-config (192.168.0.0/16) is used, and each host gets a /24 slice of it for running containers. The implementation of Flannel is quite tricky and interesting from a technical standpoint: when the data needs to cross the host boundary (going from a container to another one sitting on another host), the flannel daemon encapsulates the data on a UDP tunnel and sends it to the remote host, where it is then decoded and injected into the recipient network. Flannel uses the tunnelling facility offered by the Linux kernel: essentially, one can create a virtual network interface that looks exactly like a traditional interface from one of the two ends (e.g. “flannel0”), but from the other end is just a file (/dev/net/tun) that can be opened and packets can be read and manipulated. Let’s see if sysdig can help observe what’s happening! 

Let’s start by running sysdig with a somewhat articulated command line that will allow us to go straight to the point: we’re looking at all the network traffic activity, done by haproxy or the flannel daemon, happening on a traditional networking interface or on a tunnelling interface (/dev/net/tun). 

<pre># sysdig -pc -X '(fd.name=/dev/net/tun or fd.type=ipv4) and (proc.name=haproxy or proc.name=flanneld)'
48677 13:35:03.705139148 0 haproxy-confd (3f9e28ee47db) haproxy (4304:25) &lt; accept fd=1(&lt;4t&gt;127.0.0.1:49100-&gt;127.0.0.1:8080) tuple=127.0.0.1:49100-&gt;127.0.0.1:8080 queuepct=0
248719 13:35:03.705243712 0 haproxy-confd (3f9e28ee47db) haproxy (4304:25) &gt; recvfrom fd=1(&lt;4t&gt;127.0.0.1:49100-&gt;127.0.0.1:8080) size=8192
248720 13:35:03.705244798 0 haproxy-confd (3f9e28ee47db) haproxy (4304:25) &lt; recvfrom res=72 data=
	0x0000: 4745 5420 2f20 4854 5450 2f31 2e31 0d0a  GET / HTTP/1.1..
	0x0010: 5573 6572 2d41 6765 6e74 3a20 6375 726c  User-Agent: curl
	0x0020: 2f37 2e33 302e 300d 0a41 6363 6570 743a  /7.30.0..Accept:
	0x0030: 202a 2f2a 0d0a 486f 7374 3a6d 7961 7070   */*..Host:myapp
	0x0040: 2e6f 7267 0d0a 0d0a                      .org....
<!--4t-->
</pre>

Here we see haproxy accepting the connection coming from curl (from/to 127.0.0.1), and reading the HTTP request from it. I used the flag -X to see data buffers in hexadecimal and ASCII at the same time, since it will be useful in a second. 

<pre>248727 13:35:03.705286786 0 haproxy-confd (3f9e28ee47db) haproxy (4304:25) &gt; connect fd=2(&lt;4&gt;)
248728 13:35:03.705305408 0 haproxy-confd (3f9e28ee47db) haproxy (4304:25) &lt; connect res=-115(EINPROGRESS) tuple=192.168.69.0:52279-&gt;192.168.13.2:80
<!--4--></pre>

Once the request is received, haproxy tries to connect to 192.168.13.2, port 80 (frontend1 container), which, like we’ve seen before, should fail because that IP address belongs to a private network on another host. 

<pre>248746 13:35:03.705327335 0 &lt;na> (0d50afd028d7) flanneld (758:19) &gt; read fd=5(&lt;f>/dev/net/tun) size=8973
248747 13:35:03.705329267 0 &lt;na> (0d50afd028d7) flanneld (758:19) &lt; read res=60 data=
	0x0000: 4500 003c d44f 4000 4006 9319 c0a8 4500  E..&lt;.O@.@.....E.
	0x0010: c0a8 0d02 cc37 0050 d806 cd22 0000 0000  .....7.P..."....
	0x0020: a002 68af 6356 0000 0204 22e5 0402 080a  ..h.cV....".....
	0x0030: 0058 196d 0000 0000 0103 0307            .X.m........

248748 13:35:03.705330274 0 &lt;/na>&lt;na> (0d50afd028d7) flanneld (758:19) &gt; sendto fd=9(&lt;4u&gt;192.168.69.8:8285-&gt;10.1.1.187:8285) size=60 tuple=0.0.0.0:8285-&gt;10.1.1.187:8285
248749 13:35:03.705345645 0 &lt;/na>&lt;na> (0d50afd028d7) flanneld (758:19) &lt; sendto res=60 data=
	0x0000: 4500 003c d44f 4000 3f06 9419 c0a8 4500  E..&lt;.O@.?.....E.
	0x0010: c0a8 0d02 cc37 0050 d806 cd22 0000 0000  .....7.P..."....
	0x0020: a002 68af 6356 0000 0204 22e5 0402 080a  ..h.cV....".....
	0x0030: 0058 196d 0000 0000 0103 0307            .X.m........
&lt;/na><!--4u-->&lt;/f>&lt;/na>
</pre>

And this is where the magic happens: while haproxy is waiting for the connect() call to complete (it returned EINPROGRESS immediately because the socket was set in asynchronous mode), under the hood the kernel spits out the respective network packets on the tunnelling device file /dev/net/tun, and flannel reads, encapsulates and sends them to the remote host, using port 8285 UDP. On the other end, the remote flannel daemon reads the packet on the UDP connection and injects the payload of it (the encapsulated original packet) into its respective /dev/net/tun device, and the message is finally delivered to the frontend1 container. 

The binary "garbage" we see here is just a standard IPv4+TCP packet. I could do a little manual exercise and match the bytes according to the IPv4 and TCP header specifications (<a href="http://en.wikipedia.org/wiki/IPv4#Header" target="_blank">http://en.wikipedia.org/wiki/IPv4#Header</a> byte and <a href="http://en.wikipedia.org/wiki/Transmission_Control_Protocol#TCP_segment_structure" target="_blank">http://en.wikipedia.org/wiki/Transmission_Control_Protocol#TCP_segment_structure</a>), but I’m lazy so I’m just going to pipe that hexadecimal stream into a network analyzer such as tshark, and I’ll get the decoded packet back: 

<pre>192.168.69.0 -&gt; 192.168.13.2 TCP 74 52279→80 [SYN] Seq=0 Win=26799 Len=0 MSS=8933 SACK_PERM=1 TSval=5773677 TSecr=0 WS=128
</pre>

What happened is that the connect() is done on a TCP socket, so what we’re seeing here is the first part of the TCP three-way handshake (SYN flag) between two addresses of the private /24 networks allocated by flannel. 

<pre>248792 13:35:03.706223253 0 &lt;na> (0d50afd028d7) flanneld (758:19) &gt; recvfrom fd=9(&lt;4u&gt;192.168.69.8:8285-&gt;10.1.1.187:8285) size=8973
248793 13:35:03.706224842 0 &lt;/na>&lt;na> (0d50afd028d7) flanneld (758:19) &lt; recvfrom res=60 data=
	0x0000: 4500 003c 0000 4000 3e06 6969 c0a8 0d02  E..&lt;..@.&gt;.ii....
	0x0010: c0a8 4500 0050 cc37 c666 ef3e d806 cd23  ..E..P.7.f.&gt;...#
	0x0020: a012 688b c144 0000 0204 22e5 0402 080a  ..h..D....".....
	0x0030: 0056 ec28 0058 196d 0103 0307            .V.(.X.m....
248794 13:35:03.706225514 0 &lt;/na>&lt;na> (0d50afd028d7) flanneld (758:19) &gt; write fd=5(&lt;f>/dev/net/tun) size=60
248795 13:35:03.706241806 0 &lt;na> (0d50afd028d7) flanneld (758:19) &lt; write res=60 data=
	0x0000: 4500 003c 0000 4000 3d06 6a69 c0a8 0d02  E..&lt;..@.=.ji....
	0x0010: c0a8 4500 0050 cc37 c666 ef3e d806 cd23  ..E..P.7.f.&gt;...#
	0x0020: a012 688b c144 0000 0204 22e5 0402 080a  ..h..D....".....
	0x0030: 0056 ec28 0058 196d 0103 0307            .V.(.X.m....
&lt;/na>&lt;/f>&lt;/na><!--4u-->
</pre>

Here, the exact opposite is happening: flannel receives a packet from the remote host (encapsulated on the UDP tunnel on port 8285), and injects it in the tunnelling interface, so that haproxy, waiting on the other side, believes that the connection is really going to be established: 

<pre>192.168.13.2 -&gt; 192.168.69.0 TCP 74 80→52279 [SYN, ACK] Seq=0 Ack=1 Win=26763 Len=0 MSS=8933 SACK_PERM=1 TSval=5696552 TSecr=5773677 WS=128
</pre>

The decoded bytestream this time is a SYN+ACK packet, the second part of the TCP three-way handshake, as expected. 

<pre>248833 13:35:03.706291714 0 &lt;na> (0d50afd028d7) flanneld (758:19) &gt; read fd=5(&lt;f>/dev/net/tun) size=8973
248834 13:35:03.706292454 0 &lt;na> (0d50afd028d7) flanneld (758:19) &lt; read res=152 data=
	0x0000: 4500 0098 d451 4000 4006 92bb c0a8 4500  E....Q@.@.....E.
	0x0010: c0a8 0d02 cc37 0050 d806 cd23 c666 ef3f  .....7.P...#.f.?
	0x0020: 8018 00d2 facc 0000 0101 080a 0058 196e  .............X.n
	0x0030: 0056 ec28 4745 5420 2f20 4854 5450 2f31  .V.(GET / HTTP/1
	0x0040: 2e31 0d0a 5573 6572 2d41 6765 6e74 3a20  .1..User-Agent:
	0x0050: 6375 726c 2f37 2e33 302e 300d 0a41 6363  curl/7.30.0..Acc
	0x0060: 6570 743a 202a 2f2a 0d0a 486f 7374 3a6d  ept: */*..Host:m
	0x0070: 7961 7070 2e6f 7267 0d0a 582d 466f 7277  yapp.org..X-Forw
	0x0080: 6172 6465 642d 466f 723a 2031 3237 2e30  arded-For: 127.0
	0x0090: 2e30 2e31 0d0a 0d0a                      .0.1....

248835 13:35:03.706293252 0 &lt;/na>&lt;na> (0d50afd028d7) flanneld (758:19) &gt; sendto fd=9(&lt;4u&gt;192.168.69.8:8285-&gt;10.1.1.187:8285) size=152 tuple=0.0.0.0:8285-&gt;10.1.1.187:8285
248836 13:35:03.706301603 0 &lt;/na>&lt;na> (0d50afd028d7) flanneld (758:19) &lt; sendto res=152 data=
	0x0000: 4500 0098 d451 4000 3f06 93bb c0a8 4500  E....Q@.?.....E.
	0x0010: c0a8 0d02 cc37 0050 d806 cd23 c666 ef3f  .....7.P...#.f.?
	0x0020: 8018 00d2 facc 0000 0101 080a 0058 196e  .............X.n
	0x0030: 0056 ec28 4745 5420 2f20 4854 5450 2f31  .V.(GET / HTTP/1
	0x0040: 2e31 0d0a 5573 6572 2d41 6765 6e74 3a20  .1..User-Agent:
	0x0050: 6375 726c 2f37 2e33 302e 300d 0a41 6363  curl/7.30.0..Acc
	0x0060: 6570 743a 202a 2f2a 0d0a 486f 7374 3a6d  ept: */*..Host:m
	0x0070: 7961 7070 2e6f 7267 0d0a 582d 466f 7277  yapp.org..X-Forw
	0x0080: 6172 6465 642d 466f 723a 2031 3237 2e30  arded-For: 127.0
	0x0090: 2e30 2e31 0d0a 0d0a                      .0.1....
&lt;/na><!--4u-->&lt;/f>&lt;/na>
</pre>

After the TCP connection is successfully established, the original HTTP request is read by flannel and tunnelled to the network with the same mechanism explained earlier, of course prepending a valid IP and TCP header in this case as well. 

This explains how flannel works. However, even if everything worked fine in this example, sometimes things can go wrong. In fact, the first time I launched the two hosts, I configured the flannel network range as 10.1.0.0/16, which, oddly enough, was the same address range I used for spinning up my two hosts in AWS. I didn't immediately realize there was an address collision (I used a standard VPC that I set up a long time ago so I forgot its original address), but I could tell something was wrong: 

<pre>core@balancer ~ $ ping www.google.com
... long wait ...
</pre>

No answer, no errors. But I can run sysdig with a very similar command line like before to troubleshoot what’s happening: 

<pre># sysdig -pc -X '(fd.name=/dev/net/tun or fd.type=ipv4) and (proc.name=ping or proc.name=flanneld)'
8090 20:22:09.058323584 0 host (host) ping (2329:2329) &gt; sendto fd=4(&lt;4u&gt;10.1.88.0:39620-&gt;10.1.0.2:53) size=32 tuple=NULL
8093 20:22:09.058359052 0 &lt;na> (d55a9271f8e1) flanneld (733:17) &gt; read fd=5(&lt;f>/dev/net/tun) size=8973
8094 20:22:09.058361195 0 &lt;na> (d55a9271f8e1) flanneld (733:17) &lt; read res=60 data=
	0x0000: 4500 003c 74a6 4000 4011 5a07 0a01 5800  E..&lt;t.@.@.Z...X.
	0x0010: 0a01 0002 9ac4 0035 0028 d6e1 9317 0100  .......5.(......
	0x0020: 0001 0000 0000 0000 0377 7777 0667 6f6f  .........www.goo
	0x0030: 676c 6503 636f 6d00 0001 0001            gle.com.....
&lt;/na>&lt;/f>&lt;/na><!--4u-->
</pre>

I can see two things: first, the ping process tries to send some data to 10.1.0.2, on port 53, so this must be the DNS server needed in order to resolve www.google.com. However, I would expect this message to not go through flannel, because I’m not trying to reach a remote container on my overlay network, but a real one on the AWS network. Instead, the request seems to be routed through flannel anyway, and the decoded packet shown above is indeed a DNS request: 

<pre>10.1.88.0 -&gt; 10.1.0.2 DNS 74 Standard query 0x9317 A www.google.com
</pre>

Clearly by configuring the same address range for the physical and virtual networks I created some mess in the routing tables and now all the traffic towards 10.1.0.0/16 goes through flannel, even if it’s not meant to. Changing the address of the overlay network solved the issue, and using sysdig I was able to tell that data was flowing in the wrong network. 

There's another very interesting aspect in this scenario. This is what flannel is injecting back into the virtual interface, immediately after receiving the DNS request: 

<pre>8095 20:22:09.058362231 0 &lt;na> (d55a9271f8e1) flanneld (733:17) &gt; write fd=5(&lt;f>/dev/net/tun) size=56
8096 20:22:09.058373267 0 &lt;na> (d55a9271f8e1) flanneld (733:17) &lt; write res=56 data=
        0x0000: 4500 0038 0000 0000 0801 eec3 0a01 5800  E..8..........X.
        0x0010: 0a01 5800 0300 8afc 0000 0000 4500 003c  ..X.........E..&lt;
        0x0020: 74a6 4000 4011 5a07 0a01 5800 0a01 0002  t.@.@.Z...X.....
        0x0030: 9ac4 0035 0028 d6e1                      ...5.(..
&lt;/na>&lt;/f>&lt;/na>
</pre>

This is the decoded response: 

<pre>10.1.88.0 -&gt; 10.1.88.0 ICMP 70 Destination unreachable (Network unreachable)
</pre>

Interestingly, Flannel correctly detects that the destination address can't be reached on the overlay network, and generates a fake ICMP Destination unreachable (<a href="https://github.com/coreos/flannel/blob/master/backend/udp/proxy.c#L327" target="_blank">https://github.com/coreos/flannel/blob/master/backend/udp/proxy.c#L327</a>) that should unlock the ping process that is waiting on the other side, instead of having it wait there indefinitely (until a long timeout elapses). However, this doesn't seem to work and the ping process doesn’t report any errors, it just keeps waiting. Why does that happen? Finding out the answer to this question could be a fun project for you readers :-) 

## Troubleshooting confd 

When dealing with microservices, confd is a widely used daemon that watches a set of keys on etcd, and updates arbitrary configuration files, according to templates. In my case, confd maintains the */etc/haproxy/haproxy.cfg* configuration file. This is a snippet of the template used for haproxy.cfg: 

<pre>{{range $service := ls "/services"}}
backend {{$service}}
    balance leastconn
    {{range $upstream := ls (printf "/services/%s/upstreams" $service)}}
    server {{$upstream}} {{printf "/services/%s/upstreams/%s" $service $upstream | getv}} check
    {{end}}
{{end}}
</pre>

At runtime, confd replaces the various variables in this file with the values obtained from etcd. When troubleshooting, sysdig can be useful for watching confd regenerating the configuration file, on the fly. For example, this is what happens when I launch a new frontend unit (frontend5) with fleet, while running sysdig: 

<pre># sysdig -pc -A -c echo_fds -r coreos.scap proc.name=confd
------ Write 14B to  [haproxy-confd] [3f9e28ee47db]  /etc/haproxy/.haproxy.cfg700641834 (confd)
backend myapp
------ Write 27B to  [haproxy-confd] [3f9e28ee47db]  /etc/haproxy/.haproxy.cfg700641834 (confd)
    balance leastconn
------ Write 43B to  [haproxy-confd] [3f9e28ee47db]  /etc/haproxy/.haproxy.cfg700641834 (confd)
    server frontend1 192.168.13.2:80 check
------ Write 43B to  [haproxy-confd] [3f9e28ee47db]  /etc/haproxy/.haproxy.cfg700641834 (confd)
    <b>server frontend2 :80 check</b>
------ Write 43B to  [haproxy-confd] [3f9e28ee47db]  /etc/haproxy/.haproxy.cfg700641834 (confd)
    server frontend3 192.168.13.4:80 check
------ Write 43B to  [haproxy-confd] [3f9e28ee47db]  /etc/haproxy/.haproxy.cfg700641834 (confd)
    server frontend4 192.168.13.5:80 check
------ Write 43B to  [haproxy-confd] [3f9e28ee47db]  /etc/haproxy/.haproxy.cfg700641834 (confd)
    <b>server frontend5 :80 check</b>
</pre>

I am able to see all the I/O activity done to the temporary file during its creation, so I can catch bugs easily. In fact, I can see that frontend2 and frontend5 are not getting a valid IP address in the final configuration file, while the other three units are! Let’s see whose fault this is, by intercepting the activity between confd and etcd: 

<pre># sysdig -pc -A -c echo_fds proc.name=confd and evt.type=ipv4
------ Write 172B to  [haproxy-confd] [9768e87c382c]  10.1.1.95:56371-&gt;10.1.1.95:4001 (confd)
GET /v2/keys/haproxy-discover/services?consistent=true&recursive=true&sorted=true HTTP/1.1
Host: 10.1.1.95:4001
User-Agent: Go 1.1 package http
Accept-Encoding: gzip
------ Read 1.23KB from  [haproxy-confd] [9768e87c382c]  10.1.1.95:56371-&gt;10.1.1.95:4001 (confd)
HTTP/1.1 200 OK
Content-Type: application/json
X-Etcd-Index: 5352
X-Raft-Index: 12239
X-Raft-Term: 0
Date: Mon, 30 Mar 2015 17:29:30 GMT
Transfer-Encoding: chunked

</pre>

<pre class="wrap-text">431
{\"action\":\"get\",\"node\":{\"key\":\"/haproxy-discover/services\",\"dir\":true,\"nodes\":[{\"key\":\"/haproxy-discover/services/myapp\",\"dir\":true,\"nodes\":[{\"key\":\"/haproxy-discover/services/myapp/domain\",\"value\":\"myapp.org\",\"modifiedIndex\":5093,\"createdIndex\":5093},{\"key\":\"/haproxy-discover/services/myapp/upstreams\",\"dir\":true,\"nodes\":[{\"key\":\"/haproxy-discover/services/myapp/upstreams/frontend1\",\"value\":\"192.168.13.2:80\",\"modifiedIndex\":4544,\"createdIndex\":4544},<span style="color: red;">{\"key\":\"/haproxy-discover/services/myapp/upstreams/frontend2\",\"value\":\":80\"</span>,\"modifiedIndex\":4542,\"createdIndex\":4542},{\"key\":\"/haproxy-discover/services/myapp/upstreams/frontend3\",\"value\":\"192.168.13.4:80\",\"modifiedIndex\":4539,\"createdIndex\":4539},{\"key\":\"/haproxy-discover/services/myapp/upstreams/frontend4\",\"value\":\"192.168.13.5:80\",\"modifiedIndex\":4538,\"createdIndex\":4538},<span style="color: red;">{\"key\":\"/haproxy-discover/services/myapp/upstreams/frontend5\",\"value\":\":80\",</span>\"modifiedIndex\":5092,\"createdIndex\":5092}],\"modifiedIndex\":145,\"createdIndex\":145}],\"modifiedIndex\":145,\"createdIndex\":145}],\"modifiedIndex\":145,\"createdIndex\":145}}
0
</pre>

Here I can see the HTTP request that confd sends to etcd in order to get the full list of keys in the database, and it’s clear that the frontend2 and frontend5 addresses are not set in the database. This turned out to be a bug in my fleet unit file, there was a sort of race condition and the IP address of the unit was written to etcd before the Docker container was fully up and running, and this caused the lookup to randomly fail. I solved by adding a “sleep 2” to the frontend unit, that you can see in the <a href="https://sysdigrp2rs.wpengine.com/?p=1556" target="_blank">previous post</a>. 

Finally, confd takes care of replacing the temporary file /etc/haproxy/.haproxy.cfg700641834 with the original one at /etc/haproxy/haproxy.cfg: 

<pre># sysdig -pc proc.name=confd
...
112118 13:34:34.881802744 0 haproxy-confd (3f9e28ee47db) confd (4297:18) &gt; rename
112119 13:34:34.881817476 0 haproxy-confd (3f9e28ee47db) confd (4297:18) &lt; rename res=0 oldpath=/etc/haproxy/.haproxy.cfg700641834 newpath=/etc/haproxy/haproxy.cfg
...
</pre>

Again, this can save the trouble of logging into the container and trying to troubleshoot with traditional tools (such as a text editor in this case), that have the disadvantage of capturing a snapshot of the state of the system, whereas with sysdig we can see its entire evolution. 

## Conclusion 

In summary, CoreOS has some really cool, really powerful tech at its… core :) But troubleshooting is a pain, and troubleshooting CoreOS and containers introduces a whole new level of complexity. It’s important to use effective tools that can be: 

*   flexible and deployed in every situation
*   powerful enough to give the best visibility on what is actually happening in the system

We think that sysdig can serve this purpose quite well, and can be a good companion when dealing with the unique complexities of CoreOS. 

Check out <a href="http://www.sysdig.org/install/" target="_blank">sysdig</a> today, and try digging into CoreOS for yourself! 

And of course, if you’re looking for unified visibility across your distributed CoreOS infrastructure, you should take a look at <a href="https://sysdigrp2rs.wpengine.com/landing-quote2/?utm_source=web&utm_medium=blog&utm_campaign=COSBlog2-042815" target="_blank">Sysdig Cloud</a> ;) Sysdig Cloud is the first ever monitoring and alerting solution designed from the ground up to support containerized and CoreOS environments. 

Happy digging!