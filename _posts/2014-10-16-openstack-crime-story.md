---
ID: 536
post_title: >
  An OpenStack Crime Story solved by
  tcpdump, sysdig, and iostat
author: Ed Reckers
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/openstack-crime-story/
published: true
post_date: 2014-10-16 11:00:31
---
* * *

#### Editor's note: This post from [Lukas Pustina][1], Performance Engineer at [codecentric][2], originally appeared on the [codecentric blog][3].

#### This is the first in a new series of guest blog posts, written by real sysdig users telling their own stories in their own words. If you're interested in posting here, please [reach out][4] - we'd love to hear from you!

* * *

This is the story of how the tiniest things can sometimes be the biggest culprits. First and foremost, this is a detective story. So come and follow me on a little crime scene investigation that uncovered an incredibly counterintuitive and almost criminal default setting in Ubuntu 14.04 crippling the virtual network performance along with it's network latency.

And just in case you don’t like detective stories – and this goes out to all you nerds out there – I promise it will be worth your while, because this is also an introduction to <tt>keepalived</tt>, <tt>tcpdump</tt>, and [sysdig][5]. So sit back and enjoy!

The problems came out of the blue. We were running an [OpenStack][6] installation for the next generation [CenterDevice][7] cloud.

CenterDevice is a cloud-based document solution and our clients rely on its 24/7 availability. In order to achieve this availability, all our components are multiply redundant. For example, our load balancers are two separate virtual machines running [HAProxy][8]. Both instances manage a highly available IP address via [keepalived][9].

This system worked very well, until both load balancers became the victims of an evil crime. Both virtual machines started flapping their highly available virtual IP addresses and we were receiving alerting e-mails by the dozens, but there was no obvious change to the system that could have explained this behavior.

And this is how the story begins.

When I was called to investigate this weird behavior and answer the question of what was happening to the poor little load balancers, I started by taking a close look at the crime scene: <tt>keepalived</tt>. The following listing shows our master configuration of <tt>keepalived</tt> on virtual host loadbalancer01<a id="1"></a><sup><a href="#fn_2">1</a></sup>

<pre>% sudo cat /etc/keepalived/keepalived.conf
global_defs {
        notification_email {
                some-e-mail-address@veryimportant.people
        }
        notification_email_from loadbalancer01@protecting.the.innocent.local
        smtp_server 10.10.9.8
        smtp_connect_timeout 30
        router_id loadbalancer01
}
 
vrrp_script chk_haproxy {
        script "killall -0 haproxy"
        interval 2
        weight 2
}
 
vrrp_instance loadbalancer {
        state MASTER
        interface eth0
        virtual_router_id 205
        priority 101
        smtp_alert
        advert_int 1
        authentication {
                auth_type PASS
                auth_pass totally_secret
        }
        virtual_ipaddress {
                192.168.205.7
        }
        track_script {
                chk_haproxy
        }
}</pre>

The same configuration on our second load balancer <tt>loadbalancer02</tt> looks exactly the same, with notification email and router id changed accordingly as well as a lower priority. This all looked fine to me and it was immediately clear that <tt>keepalived.conf</tt> was not the one to blame. I needed to figure out a different reason why the two <tt>keepalived</tt> were continuously flapping the virtual IP address.

Now, it is important to understand how [VRRP][10] (the protocol <tt>keepalived</tt> uses to check the availability of its partners) works. All partners continuously exchange keep alive packets via the multicast address vrrp.mcast.net which resolves to 224.0.0.18. These packets use IP protocol number 112. If the backup does not receive these keep alive packets from the current master, it assumes the partner is dead, murdered, or has otherwise gone AWOL and takes over the virtual IP address, now acting as the new master. If the old master decides to check back in, the virtual IP address is exchanged again.

Since we were observing this back and forth exchange, I suspected the virtual network had unlawfully disregarded its responsibility. I immediately rushed to the terminal and started to monitor the traffic. This might seem easy, but it is far more complex than you think. The figure below, taken from [openstack.redhat.com][11], shows an overview of how Neutron creates virtual networks.

<img class="aligncenter wp-image-24731 size-large" src="https://blog.codecentric.de/wp-content/blogs.dir/2/files/2014/09/Neutron_architecture-1024x566.png" alt="OpenStack Neutron Architecture" width="640" height="353" /> 
Since both load balancers were running on different nodes, the VRRP packets traveled from <tt>A</tt> to <tt>J</tt> and back to <tt>Q</tt>. So where were the missing packets?

I decided to tail the packets at <tt>A</tt>, <tt>E</tt>, and <tt>J</tt>. Maybe that was where the culprit was hiding. I started <tt>tcpdump</tt> on <tt>loadbalancer01</tt>, the compute node <tt>node01</tt>, and the network controller <tt>control01</tt> looking for missing packets. And there were packets. Lots of packets. But I could not see missing packets until I stopped <tt>tcpdump</tt>:

<pre>$ tcpdump -l host vrrp.mcast.net
...
45389 packets captured
699 packets received by filter
127 packets dropped by kernel</pre>

Dropped packets by the kernel? Were these our victims? Unfortunately not.

<div>
  <tt>tcpdump</tt> uses a little buffer in the kernel to store captured packets. If too many new packets arrive before the user process <tt>tcpdump</tt> can decode them, the kernel drops them to make room for freshly arriving packets.
</div>

Not very helpful when you are trying to find where packets get, well, dropped. But there is the handy parameter <tt>-B</tt> for <tt>tcpdump</tt> that increases this buffer. I tried again:

<pre>$ tcpdump -l -B 10000 host vrrp.mcast.net</pre>

Much better, no more dropped packets by the kernel. And now, I saw something. While the VRRP packets were dumped on my screen, I noticed that sometimes there was a lag time of longer than one second between VRRP keepalives. This struck me as odd. This should not happen as the interval for VRRP packets had been set to one second in the configurations. I felt that I was on to something, but just needed to dig a bit deeper.

I let <tt>tcpdump</tt> show me the time differences between succeeding packets and detect differences bigger than one second.

<pre>$ tcpdump -i any -l -B 10000 -ttt host vrrp.mcast.net  | grep -v '^00:00:01'</pre>

[<img class="aligncenter wp-image-531 size-full" src="/wp-content/uploads/2015/09/code.png" alt="code" width="3104" height="1956" />][12]

Oh my god. Look at the screenshot above <a id="2"></a><sup><a href="#fn_2">2</a></sup>. There is a delay of more than 3.5 seconds. Of course <tt>loadbalancer02</tt> assumes his partner went MIA.

But wait, the delay already starts on <tt>loadbalancer01</tt>? I have been fooled! It is not the virtual network. It had to be the master <tt>keepalived</tt> host! But why? Why should the virtual machine hold packets back? There must be some evil hiding in this machine and I will find and face it…

* * *

After discovering the culprit might be hiding inside the virtual machine, I fetched my assistant [sysdig][5] and we ssh’ed into <tt>loadblancer01</tt>. Together, we started to ask questions - very intense, uncomfortable questions to finally get to the bottom of this.

<div>
  Sysdig is a relatively new tool that basically traces all syscalls and uses <a href="http://en.wikipedia.org/wiki/Lua_(programming_language)">Lua</a> scripts – called chisels – to derive conclusions about the captured sequence of syscalls. In general, it is like <tt>tcpdump</tt> for syscalls. Since every process that wants to do anything useful has to use syscalls, a very detailed view of your system’s behavior evolves. Sysdig is developing into a serious swiss army knife and you should give it a try - read <a href="http://draios.com/sysdig-plus-logs/">these</a> <a href="http://draios.com/ps-lsof-netstat-time-travel/">three</a> <a href="http://draios.com/hiding-linux-processes-for-fun-and-profit/">blog</a> posts to learn about sysdig’s capabilities which convinced me.
</div>

Sysdig being the good detective assistant went to work immediately and kept a transcript about all questions and answers in a file.

<pre>$ sysdig -w transcript</pre>

I was especially interested in the sendmsg syscall that actually sends network packets issued by the <tt>keepalived</tt> process.

<div>
  After a syscall is invoked by a process, the process is paused until the syscall has been finished by the kernel and only then is the execution of the process resumed.
</div>

I was interested in the invocation, which is the event direction from user process to kernel.

<pre>$ sysdig -r transcript proc.name=keepalived and evt.type=sendmsg and evt.dir='&gt;'</pre>

That stirred up quite some evidence. But I wanted to find the time gaps larger than one second. I drew Lua from my concealed holster inside my jacket and fired a quick [chisel][13] right between two sendmsg syscalls:

<pre>$ sysdig -r transcript -c find_time_gap proc.name=keepalived and evt.type=sendmsg and evt.dir='&gt;'
370964 17:01:26.375374738 (5.03543187) keepalived &gt; sendmsg fd=13(192.168.205.8-&gt;224.0.0.18) size=40 tuple=0.0.0.0:112-&gt;224.0.0.18:0</pre>

Got you! A time gap of more than five seconds turned up. I inspected the events closely looking for event number <tt>370964</tt>.

<pre>$ sysdig -r transcript proc.name=keepalived and evt.type=sendmsg and evt.dir='&gt;' | grep -C 1 370964
368250 17:01:21.339942868 0 keepalived (10400) &gt; sendmsg fd=13(192.168.205.8-&gt;224.0.0.18) size=40 tuple=0.0.0.0:112-&gt;224.0.0.18:0
370964 17:01:26.375374738 1 keepalived (10400) &gt; sendmsg fd=13(192.168.205.8-&gt;224.0.0.18) size=40 tuple=0.0.0.0:112-&gt;224.0.0.18:0
371446 17:01:26.377770247 1 keepalived (10400) &gt; sendmsg fd=13(192.168.205.8-&gt;224.0.0.18) size=40 tuple=0.0.0.0:112-&gt;224.0.0.18:0</pre>

There, in lines 1 and 2. There it was. A gap much larger than one second. Now it was clear. The perpetrator was hiding inside the virtual machine causing lags in the processing of <tt>keepalived’s</tt> sendmsg. But who was it?

After some ruling out of innocent bystanders, I found this:

<pre>$ sysdig -r transcript | tail -n +368250 | grep -v "keepalived\|haproxy\|zabbix"
369051 17:01:23.621071175 3  (0) &gt; switch next=7 pgft_maj=0 pgft_min=0 vm_size=0 vm_rss=0 vm_swap=0
369052 17:01:23.621077045 3  (7) &gt; switch next=0 pgft_maj=0 pgft_min=0 vm_size=0 vm_rss=0 vm_swap=0
369053 17:01:23.625105578 3  (0) &gt; switch next=7 pgft_maj=0 pgft_min=0 vm_size=0 vm_rss=0 vm_swap=0
369054 17:01:23.625112785 3  (7) &gt; switch next=0 pgft_maj=0 pgft_min=0 vm_size=0 vm_rss=0 vm_swap=0
369055 17:01:23.628568892 3  (0) &gt; switch next=25978(sysdig) pgft_maj=0 pgft_min=0 vm_size=0 vm_rss=0 vm_swap=0
369056 17:01:23.628597921 3 sysdig (25978) &gt; switch next=0 pgft_maj=0 pgft_min=2143 vm_size=99684 vm_rss=6772 vm_swap=0
369057 17:01:23.629068124 3  (0) &gt; switch next=7 pgft_maj=0 pgft_min=0 vm_size=0 vm_rss=0 vm_swap=0
369058 17:01:23.629073516 3  (7) &gt; switch next=0 pgft_maj=0 pgft_min=0 vm_size=0 vm_rss=0 vm_swap=0
369059 17:01:23.633104746 3  (0) &gt; switch next=7 pgft_maj=0 pgft_min=0 vm_size=0 vm_rss=0 vm_swap=0
369060 17:01:23.633110185 3  (7) &gt; switch next=0 pgft_maj=0 pgft_min=0 vm_size=0 vm_rss=0 vm_swap=0
369061 17:01:23.637061129 3  (0) &gt; switch next=7 pgft_maj=0 pgft_min=0 vm_size=0 vm_rss=0 vm_swap=0
369062 17:01:23.637065648 3  (7) &gt; switch next=0 pgft_maj=0 pgft_min=0 vm_size=0 vm_rss=0 vm_swap=0
369063 17:01:23.641104396 3  (0) &gt; switch next=7 pgft_maj=0 pgft_min=0 vm_size=0 vm_rss=0 vm_swap=0
369064 17:01:23.641109883 3  (7) &gt; switch next=0 pgft_maj=0 pgft_min=0 vm_size=0 vm_rss=0 vm_swap=0
369065 17:01:23.645069607 3  (0) &gt; switch next=7 pgft_maj=0 pgft_min=0 vm_size=0 vm_rss=0 vm_swap=0
369066 17:01:23.645075551 3  (7) &gt; switch next=0 pgft_maj=0 pgft_min=0 vm_size=0 vm_rss=0 vm_swap=0
369067 17:01:23.649071836 3  (0) &gt; switch next=7 pgft_maj=0 pgft_min=0 vm_size=0 vm_rss=0 vm_swap=0
369068 17:01:23.649077700 3  (7) &gt; switch next=0 pgft_maj=0 pgft_min=0 vm_size=0 vm_rss=0 vm_swap=0
369069 17:01:23.653073066 3  (0) &gt; switch next=7 pgft_maj=0 pgft_min=0 vm_size=0 vm_rss=0 vm_swap=0
369070 17:01:23.653078791 3  (7) &gt; switch next=0 pgft_maj=0 pgft_min=0 vm_size=0 vm_rss=0 vm_swap=0
369071 17:01:23.657069807 3  (0) &gt; switch next=7 pgft_maj=0 pgft_min=0 vm_size=0 vm_rss=0 vm_swap=0
369072 17:01:23.657075525 3  (7) &gt; switch next=0 pgft_maj=0 pgft_min=0 vm_size=0 vm_rss=0 vm_swap=0
369073 17:01:23.658678681 3  (0) &gt; switch next=25978(sysdig) pgft_maj=0 pgft_min=0 vm_size=0 vm_rss=0 vm_swap=0
369074 17:01:23.658698453 3 sysdig (25978) &gt; switch next=0 pgft_maj=0 pgft_min=2143 vm_size=99684 vm_rss=6772 vm_swap=0</pre>

The processes masked behind pids 0 and 7 were consuming the virtual CPU. 0 and 7? These low process ids indicate kernel threads! I [double checked][14] with sysdig’s supervisor, just to be sure. I was right.

The kernel, the perpetrator? This hard working, ever reliable guy? No, I could not believe it. There must be more. And by the way, the virtual machine was not under load. So what could the virtual machine’s kernel be doing … except … not doing anything at all? Maybe it was waiting for IO! Could that be? Since all hardware of a virtual machine is, well, virtual, the IO wait could only be on the baremetal level.

I left <tt>loadbalancer01</tt> for its bare metal <tt>node01</tt>. On my way, I was trying to wrap my head around what was happening. What was this all about? So many proxies, so many things happening in the shadows. So many false suspects. This is not a regular case. There is something much more vicious happening behind the scenes. I must find it…

* * *

When I arrived at baremetal host <tt>node01</tt>, hosting virtual host <tt>loadbalancer01</tt>, I was anxious to see the IO statistics. The machine must be under heavy IO load when the virtual machine’s messages are waiting for up to five seconds.

I switched on my <tt>iostat</tt> flashlight and saw this:

<pre>$ iostat
Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda              12.00         0.00       144.00          0        144
sdc               0.00         0.00         0.00          0          0
sdb               6.00         0.00        24.00          0         24
sdd               0.00         0.00         0.00          0          0
sde               0.00         0.00         0.00          0          0
sdf              20.00         0.00       118.00          0        118
sdg               0.00         0.00         0.00          0          0
sdi              22.00         0.00       112.00          0        112
sdh               0.00         0.00         0.00          0          0
sdj               0.00         0.00         0.00          0          0
sdk              21.00         0.00        96.50          0         96
sdl               0.00         0.00         0.00          0          0
sdm               9.00         0.00        64.00          0         64</pre>

Nothing? Nothing at all? No IO on the disks? Maybe my bigger flashlight <tt>iotop</tt> could help:

<pre>$ iotop</pre>

Unfortunately, what I saw was too ghastly to show here and therefore I decided to omit the screenshots of <tt>iotop</tt><a id="3"></a><sup><a href="#fn_3">3</a></sup>. It was pure horror. Six qemu processes eating the physical CPUs alive in IO.

So, no disk IO, but super high IO caused by qemu. It must be network IO then. But all performance counters show almost no network activity. What if this IO wasn’t real, but virtual? It could be the virtual network driver! It had to be the virtual network driver.

I checked the OpenStack configuration. It was set to use the para-virtualized network driver [vhost_net][15].

I checked the running qemu processes. They were also configured to use the para-virtualized network driver.

<pre>$ ps aux | grep qemu
libvirt+  6875 66.4  8.3 63752992 11063572 ?   Sl   Sep05 4781:47 /usr/bin/qemu-system-x86_64
 -name instance-000000dd -S ... -netdev tap,fd=25,id=hostnet0,vhost=on,vhostfd=27 ...</pre>

I was getting closer! I checked the kernel modules. Kernel module <tt>vhost_net</tt> was loaded and active.

<pre>$ lsmod | grep net
vhost_net              18104  2
vhost                  29009  1 vhost_net
macvtap                18255  1 vhost_net</pre>

I checked the <tt>qemu-kvm</tt> configuration and froze.

<pre>$ cat /etc/default/qemu-kvm
# To disable qemu-kvm's page merging feature, set KSM_ENABLED=0 and
# sudo restart qemu-kvm
KSM_ENABLED=1
SLEEP_MILLISECS=200
# To load the vhost_net module, which in some cases can speed up
# network performance, set VHOST_NET_ENABLED to 1.
VHOST_NET_ENABLED=0
 
# Set this to 1 if you want hugepages to be available to kvm under
# /run/hugepages/kvm
KVM_HUGEPAGES=0</pre>

<tt>vhost_net</tt> was disabled by default for <tt>qemu-kvm</tt>. We were running all packets through userspace and <tt>qemu</tt> instead of passing them to the kernel directly as <tt>vhost_net</tt> does! That’s where the lag was coming from!

I acted immediately to rescue the victims. I made the huge, extremely complicated, full 1 byte change on all our compute nodes by modifying a <tt>VHOST_NET_ENABLED=0</tt> to a <tt>VHOST_NET_ENABLED=1</tt>, restarted all virtual machines and finally, after days of constantly screaming in pain, the flapping between both load balancers stopped.

I did it! I saved them!

But I couldn’t stop here. I wanted to find out, who did that to the poor little load balancers. Who was behind this conspiracy of crippled network latency?

I knew there was only one way to finally catch the guy. I set a trap. I installed a fresh, clean, virgin Ubuntu 14.04 in a virtual machine and then, well, then I waited — for <tt>apt-get install qemu-kvm</tt> to finish:

<pre>$ sudo apt-get install qemu-kvm
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following extra packages will be installed:
  acl cpu-checker ipxe-qemu libaio1 libasound2 libasound2-data libasyncns0
  libbluetooth3 libboost-system1.54.0 libboost-thread1.54.0 libbrlapi0.6
  libcaca0 libfdt1 libflac8 libjpeg-turbo8 libjpeg8 libnspr4 libnss3
  libnss3-nssdb libogg0 libpulse0 librados2 librbd1 libsdl1.2debian
  libseccomp2 libsndfile1 libspice-server1 libusbredirparser1 libvorbis0a
  libvorbisenc2 libxen-4.4 libxenstore3.0 libyajl2 msr-tools qemu-keymaps
  qemu-system-common qemu-system-x86 qemu-utils seabios sharutils
Suggested packages:
  libasound2-plugins alsa-utils pulseaudio samba vde2 sgabios debootstrap
  bsd-mailx mailx
The following NEW packages will be installed:
  acl cpu-checker ipxe-qemu libaio1 libasound2 libasound2-data libasyncns0
  libbluetooth3 libboost-system1.54.0 libboost-thread1.54.0 libbrlapi0.6
  libcaca0 libfdt1 libflac8 libjpeg-turbo8 libjpeg8 libnspr4 libnss3
  libnss3-nssdb libogg0 libpulse0 librados2 librbd1 libsdl1.2debian
  libseccomp2 libsndfile1 libspice-server1 libusbredirparser1 libvorbis0a
  libvorbisenc2 libxen-4.4 libxenstore3.0 libyajl2 msr-tools qemu-keymaps
  qemu-kvm qemu-system-common qemu-system-x86 qemu-utils seabios sharutils
0 upgraded, 41 newly installed, 0 to remove and 2 not upgraded.
Need to get 3631 kB/8671 kB of archives.
After this operation, 42.0 MB of additional disk space will be used.
Do you want to continue? [Y/n] &lt;
...
Setting up qemu-system-x86 (2.0.0+dfsg-2ubuntu1.3) ...
qemu-kvm start/running
Setting up qemu-utils (2.0.0+dfsg-2ubuntu1.3) ...
Processing triggers for ureadahead (0.100.0-16) ...
Setting up qemu-kvm (2.0.0+dfsg-2ubuntu1.3) ...
Processing triggers for libc-bin (2.19-0ubuntu6.3) ...</pre>

And then, I let the trap snap:

<pre>$ cat /etc/default/qemu-kvm
# To disable qemu-kvm's page merging feature, set KSM_ENABLED=0 and
# sudo restart qemu-kvm
KSM_ENABLED=1
SLEEP_MILLISECS=200
# To load the vhost_net module, which in some cases can speed up
# network performance, set VHOST_NET_ENABLED to 1.
VHOST_NET_ENABLED=0
 
# Set this to 1 if you want hugepages to be available to kvm under
# /run/hugepages/kvm
KVM_HUGEPAGES=0</pre>

I could not believe it! It was Ubuntu’s own default setting. Ubuntu, the very foundation of our cloud, decided to turn <tt>vhost_net</tt> off by default, despite all modern hardware supporting it. Ubuntu was convicted and I could finally rest.

This is the end of my detective story. I found and arrested the criminal Ubuntu default setting and was able to prevent him from further crippling our virtual network latencies.

Please feel free to [contact me][16] and ask questions about the details of my journey. I’m already negotiating to sell the movie rights. But maybe there will be another season of OpenStack Crime Investigation in the future. So stay tuned!

## Footnotes

1\. <a id="fn_1"></a>All personal details like IP, E-Mail addresses etc. have been changed to protect the innocent. [⏎ ][17]

2\. <a id="fn_2"></a>Yes, you see correctly. I am running <tt>screen</tt> inside [tmux][18] . This allows me to sustain the screen tabs even during logouts. [⏎][19]

3\. <a id="fn_3"></a>Eh, and because I lost them. [⏎][20]

 [1]: https://twitter.com/drivebytesting
 [2]: http://www.codecentric.de
 [3]: https://blog.codecentric.de/en/2014/09/openstack-crime-story-solved-tcpdump-sysdig-iostat-episode-1/
 [4]: mailto:info@sysdig.com?subject=Guest%20post
 [5]: http://www.sysdig.org/
 [6]: http://www.openstack.org/ "OpenStack"
 [7]: https://www.centerdevice.de/ "CenterDevice"
 [8]: http://www.haproxy.org/ "HAProxy"
 [9]: http://www.keepalived.org/ "keepalived"
 [10]: http://en.wikipedia.org/wiki/Virtual_Router_Redundancy_Protocol "VRRP"
 [11]: https://openstack.redhat.com/Networking_in_too_much_detail "Networking in too much detail"
 [12]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/09/code.png
 [13]: https://github.com/lukaspustina/sysdig_chisels "find_time_gap"
 [14]: https://twitter.com/drivebytesting/status/507552232771182592
 [15]: http://www.linux-kvm.org/page/VhostNet "VHostNet"
 [16]: mailto:%20lukas.putina@codecentric.de
 [17]: #1
 [18]: http://tmux.sourceforge.net/ "tmux"
 [19]: #2
 [20]: #3