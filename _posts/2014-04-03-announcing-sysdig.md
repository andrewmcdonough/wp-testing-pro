---
ID: 29
post_title: 'Announcing Sysdig: a System Exploration Tool'
author: Loris Degioanni
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/announcing-sysdig/
published: true
post_date: 2014-04-03 05:32:34
---
<p dir="ltr">
  Here we are again. Launch day. Having spent a good part of the past decade working with my team on Wireshark and WinPcap, I know how satisfying it is to pour your heart into a piece of free software and watch a community grow up around it. But there’s something uniquely exciting about the day you put something new out there. Today we are launching our third ambitious Open Source project, and this time, we’re focusing on system-level monitoring and troubleshooting. We are delighted to present to you: <strong>sysdig</strong> (<a href="http://www.sysdig.org/">website</a> / <a href="https://github.com/draios/sysdig">github repo</a>).
</p>

<p dir="ltr">
  Sysdig is, for us, the natural evolution of our work with network packets. It’s the consequence of many things that, over the years, have continued to frustrate us about existing approaches.
</p>

<p dir="ltr">
  One prime example is the missing link between the network and the host. This kills me. There are tools that let you see what’s happening on the wire, and different tools that let you look at activity inside your machines. But even though the two worlds are deeply related, the picture you get today is disconnected at best. So disconnected, in fact, that the simple task of understanding <strong>which process</strong> is sending data to the network is something that has eluded a clean solution.
</p>

<p dir="ltr">
  Until now.
</p>

<p dir="ltr">
  With sysdig you can do this:
</p>

<pre>&gt; sysdig fd.type=ipv4

14:45:31.562700885 0 <strong>wget </strong>(3294) &lt; write res=113 data=GET / HTTP/1.1..User-Agent: Wget/1.14 (linux-gnu)
14:45:31.563811184 0 <strong>wget </strong>(3294) &gt; read fd=3(&lt;4t&gt;192.168.232.136:36489-&gt;192.168.232.139:80) size=313
14:45:31.563815235 0 <strong>wget</strong> (3294) &lt; read res=313 data=HTTP/1.1 302 Found..Date: Sun, 30 Mar 2014 18:45:31 GMT
14:45:31.573750284 2 <strong>dropbox </strong>(1901) &gt; recvfrom fd=22(&lt;4u&gt;192.168.74.136:17500-&gt;192.168.74.126:17500) size=65536
14:45:31.573769824 2 <strong>dropbox </strong>(1901) &lt; recvfrom res=209 data={\"host_int\": 386459914, \"version\": [1, 8] \"\"</pre>

<p dir="ltr">
  Or, if you like a simpler view, this:
</p>

<pre dir="ltr">&gt; sysdig -ctopprocs_net

Bytes Process
------------------------------
3.59KB wget
2.50KB httpd
1.94KB dropbox</pre>

<p dir="ltr">
  Another issue we found deeply frustrating is the huge drop off in quality of experience for network troubleshooting versus system-level troubleshooting. Looking at networks is done through elegant workflows that include saving the information using standard formats, exploring it with well-known filtering languages, displaying it through a de-facto standard user interface, Wireshark. Digging into system activity, on the other hand, still largely involves logging into the machine with SSH and using a plethora of dated tools with very inconsistent interfaces. And since most tools don’t offer any kind of history, you’re left struggling to reproduce the problem. Or even worse, just staring at the screen hoping that it happens again.
</p>

<p dir="ltr">
  Not anymore.
</p>

<p dir="ltr">
  With sysdig, you can easily create a trace file that you can export to a different machine:
</p>

<pre dir="ltr">sysdig -w savefile.scap</pre>

<p dir="ltr">
  Then you can slice your trace data with a filter:
</p>

<pre dir="ltr">sysdig -r savefile.scap proc.name=mysql</pre> or extract CPU usage: 

<pre dir="ltr">sysdig -r savefile.scap -ctopprocs_cpu</pre>

<p dir="ltr">
  or check where disk I/O is happening:
</p>

<pre dir="ltr">sysdig -r savefile.scap -ctopfiles</pre>

<p dir="ltr">
  or follow the data exchange on a network connection:
</p>

<pre dir="ltr">sysdig -r savefile.scap -X -cecho_fds fd.cip=192.168.0.1</pre>

<p dir="ltr">
  The information you need is all there <strong>in that file</strong>. So, before restarting a machine and hoping that the problem goes away, <strong>now you can take a trace</strong>. Just in case.
</p>

<p dir="ltr">
  We also give you a bunch of cool ways to help you find what you’re looking for in the data - like Lua scripts, which we call <a href="https://github.com/draios/sysdig/wiki/Chisels%20User%20Guide">chisels</a> (to carve up the data you unearthed.. get it?). And we intentionally designed sysdig so that more cool functionality can be added super easily, by anyone in <a href="https://groups.google.com/forum/#!forum/sysdig">the community</a> - like you!
</p>

<p dir="ltr">
  This is just the first little piece of our grand plan (so stay tuned!), but we think it offers a pretty revolutionary approach. We’ve been working really hard on this, and we are very excited to bring it to you and to hear what you think. I hope it makes your day a little better.
</p>

<p dir="ltr">
  A final plug: if you do enjoy using sysdig, we need your help spreading the word. Vote us up on <a href="https://news.ycombinator.com/">Hacker News</a> or <a href="http://www.reddit.com/r/programming/">Reddit</a>, star our <a href="https://github.com/draios/sysdig">github repo</a>, <a href="https://twitter.com/sysdig">tweet</a> at us, or write a blog post. Get involved by joining the <a href="https://groups.google.com/forum/#!forum/sysdig">official mailing list</a>. And use the link below to comment. We’d love to hear from you. Thanks!
</p>

<p dir="ltr">
  Sincerely,
</p> Loris Degioanni 

Founder and CEO