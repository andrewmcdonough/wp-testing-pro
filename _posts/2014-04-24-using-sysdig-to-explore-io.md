---
ID: 111
post_title: >
  Using Sysdig to Explore I/O with the
  “fdbytes_by” Chisel
author: Loris Degioanni
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/using-sysdig-to-explore-io/
published: true
post_date: 2014-04-24 09:00:02
---
<p dir="ltr">
  <span style="line-height: 1.5em;"><strong>fdbytes_by</strong> is one of my favorite chisels.</span>
</p>

<p dir="ltr">
  Quick aside:<a href="https://github.com/draios/sysdig/wiki/Chisels%20User%20Guide"> sysdig’s chisels</a> are embedded scripts that analyze sysdig trace files or the live event stream to perform useful actions. Chisels are written in Lua, a well known, powerful, and extremely efficient scripting language.
</p>

<p dir="ltr">
  fdbytes_by is relatively simple to use, but expressive enough to cover several interesting use cases, especially if combined with filtering. It lets you display disk, network, or any other type of I/O according to a criterium that you specify. It will update once per second if used on a live system, while it will print the full summary when applied to a file.
</p>

<p dir="ltr">
  In this post I want to share the workflow that I use to explore disk I/O usage and understand what happens in a file system. I’ll walk through an example of how to use fdbytes_by to (1) identify the amount of file I/O in a system, (2) trace that I/O to a specific directory and specific files, (3) identify the process responsible for the I/O on each file, and (4) dive into the detailed I/O activity on each file.
</p>

<h2 dir="ltr">
  The fdbytes_by Chisel
</h2>

<p dir="ltr">
  First, a basic example to show how the chisel works - here's how to display I/O activity by FD type:
</p>

<pre dir="ltr">&gt; sysdig –r trace.scap –c fdbytes_by fd.type

Bytes     fd.type
------------------------------
485.00M   file
4.63M     unix
509.92KB  pipe
99.83KB   ipv4
53.79KB   event
3.08KB    inotify</pre>

<p dir="ltr">
  We can see that this trace contains a lot of disk activity, and a bit of unix socket, pipe and network activity.
</p>

<p dir="ltr">
  Using fdbytes_by requires a basic understanding of the sysdig type system. Sysdig encodes properties (e.g. the process name) and parameters (e.g. the return value) of the events it captures using a generic type system, so that any of those properties and parameters can be used to create filters or format the output. Chisels have access to the same type system, so they can leverage it too. The –l command line flag can be used to list the available event properties:
</p>

<pre dir="ltr">&gt; sysdig –l

----------------------
Field Class: fd

fd.num            the unique number identifying the file descriptor.
fd.type           type of FD. Can be 'file', 'ipv4', 'ipv6', 'unix','pipe', 'event', 'signalfd', 'eventpoll', 'inotify' or 'signalfd'.
fd.typechar       type of FD as a single character. Can be 'f' for file, 4 for IPv4 socket, 6 for IPv6 socket, 'u' for unix socket, p for pipe, 'e' for eventfd, 's' for signalfd, 'l' for eventpoll, 'i' for inotify, 'o' for uknown.
fd.name           FD full name. If the fd is a file, this field contains the full path. If the FD is a socket, this field contain the connection tuple.
fd.directory      If the fd is a file, the directory that contains it.
fd.filename       If the fd is a file, the filename without the path.
fd.ip             matches the ip address (client or server) of the fd.
fd.cip            client IP address.
fd.sip            server IP address.
…</pre>

<p dir="ltr">
  Any of these fields can be used as an argument for fdbytes_by.
</p>

<h2 dir="ltr">
  Visualizing file I/O activity by directory
</h2>

<p dir="ltr">
  Let’s see the places in the file system where most file I/O happened. We do that by using the <strong>fd.type=file</strong> filter to limit our analysis to file I/O, and using <strong>fd.directory</strong> as the grouping criterion.
</p>

<pre dir="ltr">&gt; sysdig –r trace.scap –c fdbytes_by fd.directory "fd.type=file"

Bytes     fd.directory
------------------------------
101.89M   /tmp/
90.26M    /root/.ccache/tmp/
85.75M    /usr/include/c++/4.7.2/bits/
24.74M    /usr/include/c++/4.7.2/
23.79M    /usr/include/
15.17M    /root/agent/build/release/userspace/libsanalyzer/CMakeFiles/sanalyzer.dir/
10.05M    /usr/include/bits/
9.82M     /root/agent/build/release/userspace/libsanalyzer/CMakeFiles/sanalyzer.dir/root/sysdig/userspace/libsinsp/
7.90M     /root/.ccache/d/e/
7.54M     /root/agent/userspace/libsanalyzer/
7.44M     /usr/include/c++/4.7.2/x86_64-redhat-linux/bits/</pre>

<p dir="ltr">
  Looks like there’s some serious compiling going on on this machine! A lot of the activity, in fact, happens in directories containing C or C++ include files. But the top directory, with 101MB of I/O, is /tmp. What’s happening in there?
</p>

<h2 dir="ltr">
  Visualizing file I/O activity for each file in a directory
</h2>

<p dir="ltr">
  To find out which files have been accessed in /tmp, we use a filter that keeps just the activity in /tmp, and we group by file name:
</p>

<pre dir="ltr">&gt; sysdig –r trace.scap –c fdbytes_by fd.filename "fd.directory=/tmp/"

Bytes     fd.filename
------------------------------
13.64M    ccJvb4Mi.s
9.33M     cc4pbJkV.s
6.72M     ccx9zir6.s
5.95M     cc9ZG6af.s
4.05M     ccTSZOV9.s
4.04M     cc6IZVan.s
3.88M     ccd7wm9E.s
3.81M     ccG0IloP.s
3.73M     ccxV9PLa.s
3.61M     ccRxYpyU.s</pre>

<p dir="ltr">
  A little trick here: <strong>fd.filename</strong> shows just the name of the file, with no path. If you are interested in the full path, use <strong>fd.name</strong>.
</p>

<p dir="ltr">
  Based on their extension, those definitely look like compiler intermediate files, so they are probably related to the compiling happening on the machine. But let’s make sure.
</p>

<h2 dir="ltr">
  Identifying file I/O activity by process name
</h2>

<p dir="ltr">
  Here’s how I can find out which processes accessed these files:
</p>

<pre dir="ltr">&gt; sysdig –r trace.scap –c fdbytes_by proc.name "fd.directory=/tmp/ and fd.filename contains .s"

Bytes     proc.name
------------------------------
50.57M    as
49.03M    cc1plus
1.54M     cc1
1B        httpd</pre>

<p dir="ltr">
  Notice how this time the filter accepts only files in /tmp whose name contains “.s”. And, as expected, the result shows that the C++ compiler and assembler are the culprits.
</p>

<h2 dir="ltr">
  Exploring detailed file I/O activity for a file
</h2>

<p dir="ltr">
  Need more details? We can find out exactly what was going on with those files by displaying the <strong>full</strong> I/O activity that they were involved with. Let’s pick for example the file on top of the list, ccJvb4Mi.s:
</p>

<pre dir="ltr">&gt; sysdig -A –r trace.scap –c echo_fds "fd.filename=ccJvb4Mi.s"

------ Write 4.00KB to /tmp/ccJvb4Mi.s
.file"chisel.cpp"
.text
.Ltext0:
.section.text._ZNSt9exceptionC2Ev,"axG",@p

------ Write 4.00KB to /tmp/ccJvb4Mi.s
_ZdlPvS_:
.LFB382:
.loc 3 117 0
.cfi_startproc
pushq%rbp
.cfi_def_cfa_off

------ Write 4.00KB to /tmp/ccJvb4Mi.s
x), %rax
testq%rax, %rax
je.L26
.loc 5 250 0
movq-8(%rbp), %rax
movq411

…</pre>

<p dir="ltr">
  Notice how the first write on the file includes the name of a file, chisel.cpp, which is part of the sysdig source code repository! We can safely claim that the machine disk I/O, including the one in /tmp, was caused by somebody compiling sysdig. I guess that would be me!
</p>

<p dir="ltr">
  This is of course just an example, but the takeaway here is the workflow around fdbytes_by. Once you get used to it, it will come pretty naturally and you can apply it to identifying disk bottlenecks for your servers, your databases and a bunch of other stuff.
</p> My next post will cover using fdbytes_by to explore network usage, so stay tuned!