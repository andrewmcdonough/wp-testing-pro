---
ID: 902
post_title: Interpreting Sysdig Output
author: Loris Degioanni
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/interpreting-sysdig-output/
published: true
post_date: 2014-12-16 11:20:19
---
**  
**
The most powerful system troubleshooting tools all tend to have one trait in common: a non-negligible learning curve. As the saying goes, if you want to drive a race car, you have to learn to drive stick*. Now to be fair, we designed sysdig from the ground up to be as intuitive and straightforward to use as possible. But to get the most out of the tool, it definitely helps to have an understanding of how system level information is presented in sysdig output. In this post, I will walk through some basic operating system elements and try to explain how they are represented by sysdig, including:

*   Processes and Threads
*   File Descriptors
*   Events, System Calls and Switches

## Sysdig output basics

Running sysdig without any filters will produce output that looks like this *(colors added for easier explanation, below)*:

<span style="color: #ff0000;">1</span> <span style="color: #a65c0d;">01:40:19.601363716</span> <span style="color: #0000ff;">1</span> <span style="color: #800080;">httpd</span> <span style="color: #ff9900;">(7513)</span> <span style="color: #ff00ff;">></span> <span style="color: #339966;">accept</span>

<span style="color: #ff0000;">2</span><span style="color: #a65c0d;"> 01:40:19.601374197</span> <span style="color: #0000ff;">1</span> <span style="color: #800080;">httpd</span> <span style="color: #ff9900;">(7513)</span> <span style="color: #ff00ff;"><</span> <span style="color: #339966;">accept</span> fd=14(<4t>127.0.0.1:39175->127.0.0.1:80) tuple=127.0.0.1:39175->127.0.0.1:80 queuepct=0

<span style="color: #ff0000;">3</span> <span style="color: #a65c0d;">01:40:19.601506564</span> <span style="color: #0000ff;">1</span> <span style="color: #800080;">httpd</span> <span style="color: #ff9900;">(7513)</span> <span style="color: #ff00ff;">></span> <span style="color: #339966;">read</span> fd=14(<4t>127.0.0.1:39175->127.0.0.1:80) size=8000

<span style="color: #ff0000;">4</span> <span style="color: #a65c0d;">01:40:19.601512497</span> <span style="color: #0000ff;">1</span> <span style="color: #800080;">httpd</span> <span style="color: #ff9900;">(7513)</span> <span style="color: #ff00ff;"><</span> <span style="color: #339966;">read</span> res=85 data=GET /textfile.txt HTTP/1.1..User-Agent: curl/7.35.0..Host: 127.0.0.1..Accept: */

<span style="color: #ff0000;">5</span> <span style="color: #a65c0d;">01:40:19.601516976</span> <span style="color: #0000ff;">0</span> <span style="color: #800080;">httpd</span> <span style="color: #ff9900;">(3750)</span> <span style="color: #ff00ff;">></span> <span style="color: #339966;">switch</span> next=0 pgft_maj=0 pgft_min=522 vm_size=350196 vm_rss=9304 vm_swap=0

<span style="color: #ff0000;">6</span><span style="color: #a65c0d;"> 01:40:19.601661779</span> <span style="color: #0000ff;">1</span> <span style="color: #800080;">httpd</span> <span style="color: #ff9900;">(7513)</span> <span style="color: #ff00ff;">></span> <span style="color: #339966;">open</span>

<span style="color: #ff0000;">7</span> <span style="color: #a65c0d;">01:40:19.601668329</span> <span style="color: #0000ff;">1</span> <span style="color: #800080;">httpd</span> <span style="color: #ff9900;">(7513)</span> <span style="color: #ff00ff;"><</span> <span style="color: #339966;">open</span> fd=15(<f>/opt/lampp/htdocs/textfile.txt) name=/opt/lampp/htdocs/textfile.txt flags=1(O_RDONLY) mode=0

<span style="color: #ff0000;">8</span> <span style="color: #a65c0d;">01:40:19.601699335</span> <span style="color: #0000ff;">1</span> <span style="color: #800080;">httpd</span> <span style="color: #ff9900;">(7513)</span> <span style="color: #ff00ff;">></span> <span style="color: #339966;">read</span> fd=14(<4t>127.0.0.1:39175->127.0.0.1:80) size=8000

<span style="color: #ff0000;">9</span> <span style="color: #a65c0d;">01:40:19.601701560</span> <span style="color: #0000ff;">1</span> <span style="color: #800080;">httpd</span> <span style="color: #ff9900;">(7513)</span> <span style="color: #ff00ff;"><</span> <span style="color: #339966;">read</span> res=-11(EAGAIN) data=

<span style="color: #ff0000;">10</span> <span style="color: #a65c0d;">01:40:19.601855764</span> <span style="color: #0000ff;">1</span> <span style="color: #800080;">httpd</span> <span style="color: #ff9900;">(7513)</span> <span style="color: #ff00ff;">></span> <span style="color: #339966;">close</span> fd=15(<f>/opt/lampp/htdocs/textfile.txt)

<span style="color: #ff0000;">11</span><span style="color: #a65c0d;"> 01:40:19.601857490</span> <span style="color: #0000ff;">1</span> <span style="color: #800080;">httpd</span> <span style="color: #ff9900;">(7513)</span> <span style="color: #ff00ff;"><</span> <span style="color: #339966;">close</span> res=0** **

The fields printed by sysdig are:

*   <span style="color: #ff0000;">Incremental event number</span>
*   <span style="color: #a65c0d;">Event timestamp</span> - customize this with the -t command line flag ([more info][1])
*   <span style="color: #0000ff;">CPU ID</span>
*   <span style="color: #800080;">Command name</span>
*   <span style="color: #ff6600;">Thread ID</span>
*   <span style="color: #ff00ff;">Event direction</span> - ‘>’ means ‘enter’, while ‘<’ means ‘exit’
*   <span style="color: #339966;">Event type</span>
*   Event arguments

Now let’s dive into this output a little further and see how some fundamental operating system elements are represented here by sysdig.

## Processes and Threads

Threads are the core execution unit for the operating system and, as a consequence, for sysdig as well.Multiple threads can exist within the same process or command and share resources such as memory. This is clearly shown in the capture snippet above: note how all the events in the snippet are generated by the httpd command, but one of them (number 5) comes from a different thread, on a different CPU.

The thread ID (aka TID) is the basic identifier that you want to follow when tracking execution activity in your machine. You do that by just looking at the TID number, or by filtering out the noise with a command line like this one:

<pre>&gt; sysdig thread.tid=7513</pre>

which will preserve only the execution flow for thread 7513.

Threads live inside processes, which are identified by a process ID (or PID). Most of the processes running on an average Linux box are single threaded, and in that case thread.tid is the same as proc.pid. Filtering by proc.pid is useful to observe how different threads interact with each other inside a process.

And here’s a couple of little tricks: if you want to see only events for single threaded processes, you can use this command line:

<pre>&gt; sysdig proc.nchilds=0</pre>

While if you want to trace a specific user session or script, you can use:

<pre>&gt; sysdig proc.apid = X</pre>

...where X is the pid of the shell running the script

## File Descriptors

You may have noticed that several of the events above have an argument called “fd”.

If a thread ID identifies a flow of execution, a file descriptor (FD) identifies an input/output flow.

In unix, a lot of stuff is abstracted as a file, including:

*   files
*   network connections (sockets)
*   standard input, standard output, and standard error
*   pipes
*   timers
*   signals

An FD is a numeric ID that uniquely identifies a file **inside a process**. This means that you can see the same FD number used more than once, but it will have to be in different processes. By following a process/FD combination, you can track specific I/O activity, and as usual you can let a filter help you:

<pre>&gt; sysdig proc.name=httpd and fd.num=14</pre>

Sysdig is also nice enough to resolve FD numbers into human readable strings. For example, in the snippet above, notice how FD 14 is represented as *<4t>127.0.0.1:39175->127.0.0.1:80*, which means:

*   this is an IPv4 socket (*4*)
*   the socket L4 protocol is TCP (*t*)
*   the connection has been established by port 39175 on 127.0.0.1
*   the destination of the connection is port 80 on 127.0.0.1

All of this information can conveniently be used in filters to isolate interesting activity. For example: 

<pre>&gt; sysdig fd.type=ipv4
&gt; sysdig fd.l4proto=tcp
&gt; sysdig fd.sip=127.0.0.1
&gt; sysdig fd.sport=39157</pre>

## Events

Most of the events that sysdig captures and displays are system calls. The system call interface includes a number of ‘functions’ that the operating system exports to the applications running on top of it to do stuff like open files, create network connections, read and write from an FD, and so on.

Usually, you can get more information about one of the sysdig events by typing:

<pre>&gt; man 2 &lt;event_name&gt;</pre>

in a terminal or in google. Be aware, however, that differently from a tool like strace, sysdig **doesn't mirror the system calls’ syntax with absolute precision**.** This means that, even if the system call manual page will give you a very good indication of what the event does and what its arguments mean, there might be discrepancies between the original system call and the corresponding sysdig event. If you want to know what the arguments are for any event, type:

<pre>&gt; sysdig -L</pre>

As you’ll see if you run this command, sysdig exports many events. However, some of them tend to be more useful for debugging or exploration. For example:

*   clone() and execve() give you insight into process creation and command execution.
*   open(), close(), and the FD read and write functions offer visibility on disk I/O.
*   socket(), connect(), and accept() give insight into network activity.

If you are interested in what these system calls do and how to interpret them, come back to the sysdig blog tomorrow for a whole post focused on that.

In the meantime, there’s an event that I want to mention because people sometimes get confused about it: <tt>switch</tt>.

## The Switch Event

Some people don’t realize that sysdig is not limited to capturing system calls. Sysdig taps into the kernel to report other types of events as well. The <tt>switch</tt> event, for example, is generated every time there is a context switch, i.e. when the process scheduler puts a thread to sleep to execute another one. Observing scheduler activity is useful for scenarios such as determining when a process/thread runs with high granularity, debugging process synchronization issues, and monitoring when a thread is migrated to a different CPU.

<tt>switch</tt> is not a system call, so you won’t find a manual page for it. It’s also one of the few sysdig events that doesn’t have an exit event, so you will only see enter as a direction. <tt>switch</tt> can also be annoying at times, because sysdig tends to print many of them even on systems with relatively low activity. Fortunately, it’s easy to filter it out if you don’t need it:

<pre>&gt; sysdig evt.type!=switch</pre>

## Conclusion

Processes, threads, file descriptors, system calls and context switches are some of the fundamental elements of the Linux operating system. Hopefully it’s more clear now how sysdig represents information about these elements, and how you can use sysdig to explore your system in some pretty powerful ways. ;) If you’d like to dive into even more information on sysdig, be sure to check out the [sysdig User Guide][2] on our wiki.

And now that you understand what all that sysdig output on your screen means, it’s time to explore which system calls to monitor and what useful information they hide. More on this in my next post tomorrow - so be sure to come back!

And remember to [follow us on twitter][3] for cool sysdig commands with the #DigoftheDay.

* * *

**This might not be a saying per se, but my friend Gerald Combs did just say it… so I guess it’s a saying now. Thanks Gerald! :)*

***In case you're wondering, the main reason for this is performance. sysdig’s primary design goal is efficiency, so you can use it in production with minimal overhead. This puts constraints on the order the arguments are returned, to minimize page fault probability. In addition, sysdig adds some useful arguments to some of the system calls. For example, memory-related events like brk or mmap return the total process memory usage, which tends to be handy when troubleshooting memory-related issues. *

 [1]: https://github.com/draios/sysdig/wiki/Sysdig-Manual
 [2]: https://github.com/draios/sysdig/wiki/Sysdig%20User%20Guide
 [3]: https://twitter.com/sysdig