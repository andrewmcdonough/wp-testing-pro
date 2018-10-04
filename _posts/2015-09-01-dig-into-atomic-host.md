---
ID: 2018
post_title: Dig into Atomic Host!
author: Gianluca Borello
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/dig-into-atomic-host/
published: true
post_date: 2015-09-01 07:00:40
---
The latest release of <a href="http://www.sysdig.org/" target="_blank">sysdig</a> contains some <a href="https://sysdigrp2rs.wpengine.com/announcing-sysdig-0-1-103/" target="_blank">exciting new features</a>, and in this blog post I want to focus on one specifically that was requested by users running containers: we are excited to expand the sysdig support for minimal container-oriented Linux distributions, and are adding **RHEL Atomic Host**, **CentOS Atomic Host** and **Fedora Atomic Host**, in addition to the already supported **CoreOS**. 

Also, since our <a href="https://sysdigrp2rs.wpengine.com/" target="_blank">Sysdig Cloud</a> agent is directly based on the same sysdig components from this release, now you can fully monitor an entire cluster of Atomic Hosts just by using the Sysdig Cloud Docker container. 

## Basics 

Atomic Hosts are minimal versions of a traditional distribution, designed to run almost exclusively Docker containers. There are several advantages to building the operating system in this way, but monitoring / troubleshooting / visibility is definitely not one of them: both the host OS and your containers will be missing pretty much every troubleshooting tool that you’re used to having. 

Now you can run sysdig directly in the Atomic host, and in seconds you’ll have full visibility into the host and all of your containers! 

For example, this is how you run sysdig inside a CentOS Atomic Host: 

<pre>$ cat /etc/redhat-release
CentOS Linux release 7.1.1503 (Core)
<span style="color: red;">$ sudo docker run -i -t --name sysdig --privileged -v /var/run/docker.sock:/host/var/run/docker.sock -v /dev:/host/dev -v /proc:/host/proc:ro -v /boot:/host/boot:ro -v /lib/modules:/host/lib/modules:ro -v /usr:/host/usr:ro sysdig/sysdig</span>
* Trying to find precompiled sysdig-probe for 3.10.0-229.4.2.el7.x86_64
Found kernel config at /host/usr/lib/ostree-boot/config-3.10.0-229.4.2.el7.x86_64
* Trying to download precompiled module from https://s3.amazonaws.com/download.draios.com/stable/sysdig-probe-binaries/sysdig-probe-0.1.103-x86_64-3.10.0-229.4.2.el7.x86_64-efcc01a8f6f24861684bd7af149e5523.ko
Download succeeded, loading module
root@700d4958f3cd:/# <span style="color: red;">sysdig -pc</span>
7 21:31:42.603553658 0 sysdig (700d4958f3cd) sysdig (12166:529) > switch next=24 pgft_maj=2 pgft_min=1101 vm_size=44064 vm_rss=3128 vm_swap=0
75 21:31:46.440599093 0 host (host) docker (4714:4714) &lt; read res=1484 data=7 21:31:46.410324248 0 sysdig (700d4958f3cd) sysdig (12167:530) > switch next=24
76 21:31:46.440601192 0 host (host) docker (4714:4714) > futex addr=11D2AC0 op=1(FUTEX_WAKE) val=1
...
</pre>

Once started, you can use sysdig the traditional way, inspecting all the other containers running on your Atomic Host. 

## How it works 

Sysdig is composed of two parts: the kernel driver, **sysdig-probe**, responsible for extracting system events from the kernel, and the userspace programs, sysdig and csysdig. 

Sysdig-probe, being a kernel module, must be compiled against the exact same kernel that is running in your host. Usually, for traditional Linux distributions, this task is easily accomplished by installing a C compiler and the headers for the running kernel. Generally, this is all automated using the dkms framework, which sysdig depends on. 

However, when you run sysdig inside a Docker container, you are essentially running a completely different runtime inside the container, and getting the kernel headers of the host runtime is not always an easy task. Previously, depending on the host OS, there have been two general approaches we’ve taken to solving this problem for sysdig: 
1.  Having the user manually install the kernel headers for the running kernel in the host and bind mount them when starting the Docker container. 
2.  Precompiling sysdig-probe for every possible kernel version on our side, and downloading it at runtime into the container during the bootstrap phase. 

For some distributions, such as CoreOS and Atomic Host, the first option is not possible, because the kernel headers are not available in the host, since it completely lacks a package manager. With this <a href="https://sysdigrp2rs.wpengine.com/announcing-sysdig-0-1-103/" target="_blank">new 0.1.103 release of sysdig</a>, we are now precompiling all the sysdig-probe kernel modules for all the kernels officially released by RHEL, Fedora, CentOS and Ubuntu. This has the following benefits: 
1.  It is now possible to use the sysdig Docker container on distributions that don’t ship kernel headers (such as Atomic Host). 
2.  For distributions that normally ship kernel headers (e.g. Ubuntu, CentOS, Fedora, ... in their traditional versions), installing the kernel headers in the host is most likely not necessary anymore*, simplifying the whole sysdig installation process. 

## Conclusion 

We keep iterating towards making sysdig more and more useful when it comes to troubleshooting containers, and we hope that little improvements like these will help the entire container ecosystem to move forward. And as always, more good stuff is coming soon. <a href="http://www.sysdig.org/install/" target="_blank">Happy digging!</a> 

* We say most likely because at this moment we are not precompiling all kernel flavors (e.g. lowlatency, realtime, pae), but if you find a distribution/kernel flavor that you’d like to see supported, reach out on github and we’ll likely make it happen.