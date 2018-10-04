---
ID: 268
post_title: 'Sysdig + Logs: Advanced Log Analysis Made Easy'
author: Loris Degioanni
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-plus-logs/
published: true
post_date: 2014-08-07 10:20:41
---
Log collection and analysis is one the most powerful tools in the hands of developers and operations teams. Inspecting logs is useful in a number of areas, including security, monitoring, debugging and troubleshooting. Often, however, dealing with logs is frustrating, because they give just a hint about a problem and don’t provide enough context to understand what the problem actually is or how to fix it. Addressing the inherent shortcomings of logs has always been one of our many goals with sysdig. And the latest release of sysdig, in particular, adds some pretty cool functionality for log collection, inspection and analysis. 

<span><br /></span>
This blog post will demonstrate a couple interesting methods for using sysdig with logs, including:

*   Tailing all of the machine log files with a single command line, without the need to configure anything
*   Easily correlating log entries with the relevant system activity
*   Troubleshooting a failed PHP request with sysdig + logs

Note: this post will likely make more sense if you have a basic familiarity with sysdig. If you’ve never heard of sysdig, I suggest you start with the [website][1] or the [wiki][2].

## Analyzing syslog with sysdig

Since sysdig sees all the I/O on the system, sysdig captures all syslog activity by default. A sysdig chisel called spy_syslog is available to easily isolate and display that syslog activity. For example, if I run spy_syslog while manually generating a bit of syslog noise, I get an output like this:

<pre>$ sudo sysdig -c spy_syslog
<span style="color: #ff0000;">local3.emerg logger[14024] Jul 30 16:35:53 root: This is an emergency message
local3.alert logger[14025] Jul 30 16:35:53 root: This is an alert message
local3.crit logger[14026] Jul 30 16:35:53 root: This is a critical message
local3.err logger[14027] Jul 30 16:35:53 root: This is an error message</span>
<span style="color: #eeaa00;">local3.warn logger[14028] Jul 30 16:35:53 root: This is a warning message</span>
<span style="color: #339966;">local3.notice logger[14029] Jul 30 16:35:53 root: This is a notice message</span>
<span style="color: #339966;">local3.info logger[14030] Jul 30 16:35:53 root: This is an info message</span>
<span style="color: #339966;">local3.debug logger[14031] Jul 30 16:35:53 root: This is a debug message</span>
<span style="color: #ff0000;">user.err python[14032] root: This is an error message from ptyhon</span>
<span style="color: #eeaa00;">user.warn python[14032] root: This is a warning message from ptyhon</span>
<span style="color: #ff0000;">user.debug python[14032] root: This is a debug message from ptyhon</span></pre>

As you can see, sysdig displays each syslog message as it occurs on my system (in this case, logs from my ‘logger’ and ‘python’ processes), and it uses color to indicate the severity of the message. And as always, this chisel is integrated with the sysdig filtering language, so that I can dig into syslog activity using arbitrary expressions. For example, I can capture only messages with high severity coming from the process ‘logger’:

<pre>$ sudo sysdig -c spy_syslog 'syslog.severity &lt; 4 and proc.name=logger'
<span style="color: #ff0000;">local3.emerg logger[14024] Jul 30 16:35:53 root: This is an emergency message
local3.alert logger[14025] Jul 30 16:35:53 root: This is an alert message
local3.crit logger[14026] Jul 30 16:35:53 root: This is a critical message
local3.err logger[14027] Jul 30 16:35:53 root: This is an error message</span></pre>

Also, as always, this will work on a live system, but also on a trace file that I captured previously. By the way, if you are interested in syslog messages only, this command will create a compact trace file that only includes those:

<pre>$ sudo sysdig -F -w trace.scap evt.is_syslog=true</pre>

## Analyzing application logs with sysdig

Another chisel that is very useful for log spelunking is spy_logs. It works in a similar way to spy_syslog, but instead of showing messages written to /var/log/messages, it shows messages that are written to files that contain “.log” or “\_log” in their name. (Note, if you want to tune this, just edit the FILE\_FILTER constant at the beginning of the chisel Lua code). spy_logs automatically unpacks multi-line writes and renders each of them nicely on the screen, with some basic pattern-matching-based coloring.

Think about it as a filterable super-tail that shows all the log files in the system, with no configuration required. Just run it, and you get **all of the logs in a single stream**. It’s sort of magic. Don’t believe me? Here, for example, I ran the chisel for a minute or two on one of my VMs:

<pre>$ sudo sysdig -c spy_logs
<span style="color: #339966;">httpd /opt/lampp/logs/error_log [Wed Jul 30 20:07:49.180019 2014] [suexec:notice] [pid 28262] AH01232: suEXEC mechanism enabled (wrapper: /opt/lampp/bin/suexec)</span>
<span style="color: #339966;">httpd /opt/lampp/logs/error_log [Wed Jul 30 20:07:49.389739 2014] [auth_digest:notice] [pid 28263] AH01757: generating secret for digest authentication ...</span>
<span style="color: #eeaa00;">httpd /opt/lampp/logs/error_log [Wed Jul 30 20:07:50.045576 2014] [ssl:warn] [pid 28263] AH01906: RSA server certificate is a CA certificate (BasicConstraints: CA == TRUE !?)</span>
<span style="color: #eeaa00;">httpd /opt/lampp/logs/error_log [Wed Jul 30 20:07:50.045645 2014] [ssl:warn] [pid 28263] AH01909: RSA certificate configured for www.example.com:443 does NOT include an ID which matches the server name</span>
<span style="color: #339966;">httpd /opt/lampp/logs/error_log [Wed Jul 30 20:07:50.045720 2014] [lbmethod_heartbeat:notice] [pid 28263] AH02282: No slotmem from mod_heartmonitor</span>
<span style="color: #339966;">httpd /opt/lampp/logs/error_log [Wed Jul 30 20:07:50.943101 2014] [mpm_prefork:notice] [pid 28263] AH00163: Apache/2.4.4 (Unix) OpenSSL/1.0.1e PHP/5.4.16 mod_perl/2.0.8-dev Perl/v5.16.3 configured -- resuming normal operations</span>
<span style="color: #ff0000;">httpd /opt/lampp/logs/error_log [Wed Jul 30 20:07:50.943188 2014] [core:notice] [pid 28263] AH00094: Command line: '/opt/lampp/bin/httpd -E /opt/lampp/logs/error_log -D SSL -D PHP'</span>
<span style="color: #339966;">httpd /opt/lampp/logs/access_log 127.0.0.1 - - [30/Jul/2014:20:08:47 -0400] "GET /xampp/splash.php HTTP/1.1" 200 1321</span>
<span style="color: #339966;">httpd /opt/lampp/logs/access_log 127.0.0.1 - - [30/Jul/2014:20:08:48 -0400] "GET /xampp/xampp.css HTTP/1.1" 304 -</span>
<span style="color: #ff0000;">cupsd /var/log/cups/access_log localhost - - [30/Jul/2014:20:14:03 -0400] "POST / HTTP/1.1" 200 183 Renew-Subscription client-error-not-found</span>
<span style="color: #339966;">redis-server /var/log/redis/redis.log [</span>
<span style="color: #339966;">redis-server /var/log/redis/redis.log 27066</span>
<span style="color: #339966;">redis-server /var/log/redis/redis.log | signal handler] (</span>
<span style="color: #339966;">redis-server /var/log/redis/redis.log 1406766138</span>
<span style="color: #339966;">redis-server /var/log/redis/redis.log )</span>
<span style="color: #339966;">redis-server /var/log/redis/redis.log Received SIGTERM, scheduling shutdown...</span>
<span style="color: #339966;">redis-server /var/log/redis/redis.log [27066] 30 Jul 20:22:18.877 # User requested shutdown...</span>
<span style="color: #339966;">redis-server /var/log/redis/redis.log [27066] 30 Jul 20:22:18.877 * Saving the final RDB snapshot before exiting.</span></pre>

The first item in each line is the process name, the second is the name of the log file, and the rest of the line contains the actual message.

As always, this chisel can be applied to a live system (in that case it will behave very similarly to tail) or to a trace file that has been captured previously. And as always you can use filters. This command line, for example, will capture log writes made by httpd and containing the word GET.

<pre>$ sudo sysdig -c spy_logs proc.name=httpd and evt.buffer contains GET
<span style="color: #339966;">httpd /opt/lampp/logs/access_log 127.0.0.1 - - [30/Jul/2014:20:28:07 -0400] "GET /xampp/splash.php HTTP/1.1" 200 1321</span>
<span style="color: #339966;">httpd /opt/lampp/logs/access_log 127.0.0.1 - - [30/Jul/2014:20:28:11 -0400] "GET /xampp/xampp.css HTTP/1.1" 304 -</span>
<span style="color: #339966;">httpd /opt/lampp/logs/access_log 127.0.0.1 - - [30/Jul/2014:20:28:11 -0400] "GET /xampp/img/blank.gif HTTP/1.1" 304 -</span>
<span style="color: #339966;">httpd /opt/lampp/logs/access_log 127.0.0.1 - - [30/Jul/2014:20:28:11 -0400] "GET /xampp/img/xampp-logo.jpg HTTP/1.1" 304 -</span></pre>

## Using logs and system events to troubleshoot a failed HTTP request



Now let’s get into the cool stuff.



Sysdig really starts to shine when the log collection capabilities presented in the previous sections are married with sysdig’s innate ability to do deep system inspection. For example, the spy_logs chisel has a very powerful feature: **it can save a snapshot of the activity of the thread that wrote a log event, around the time when the event was written.**



As usual, I’ll use a real world example to show you what I mean.



One of the pages in my web server is failing with a generic “database error”. I want to understand what’s causing the failure. And, by the way, I have to admit I don’t really know PHP.

[<img class="aligncenter wp-image-273" src="/wp-content/uploads/2015/08/Screen-Shot-2014-08-04-at-1.30.57-PM.png" alt="Screen Shot 2014-08-04 at 1.30.57 PM" width="680" height="556" />][3] 

I’m going to start by taking a sysdig capture while I load the page, hoping that the problem will show up in some log. I start the capture with:



<pre>$ sudo sysdig -s2000 -w system.scap</pre>



...and stop the capture with CTRL+C after refreshing my browser window. Now I have a full snapshot of my system while the problem happened. I start by taking a look at the log activity:



<pre>$ sysdig -r system.scap -c spy_logs
<span style="color: #ff0000;">httpd /opt/lampp/logs/php_error_log [31-Jul-2014 02:46:33] Database error</span>
<span style="color: #339966;">httpd /opt/lampp/logs/access_log 127.0.0.1 - - [30/Jul/2014:20:46:32 -0400] "GET /userstats.php HTTP/1.1" 200 72</span></pre>



Bingo. The second line is the page access log, and confirms that the URL is the right one, while the first line is the error we’re looking for: “Database error”. Not surprisingly, it’s not very helpful. And normally, that is all you have, which is the typical issue with logs. This time, however, I have a shiny new sysdig trace!



Let’s start by using that powerful feature I mentioned above: I’ll isolate the relevant system activity around the time of the failed request into a separate sysdig file. Here’s the command:



<pre>$ sysdig -r system.scap -c spy_logs "request.scap 1000" "evt.buffer contains Database"
<span style="color: #ff0000;">httpd /opt/lampp/logs/php_error_log [31-Jul-2014 02:46:33] Database error</span>
Writing events for 1 log entries</pre>



Since this command is a little bit more advanced, let’s look at it piece by piece:



*   -r system.scap indicates the trace file that we want to read from
*   -c spy_logs "request.scap 1000" means: for each event selected, save 1000ms of system activity before it happened and 1000ms of activity after it happened, but just for the thread that generated the event, and save it to request.scap
*   “evt.buffer contains Database” is our filter for selecting events, which we use to isolate the specific log entry we’re interested in (and in fact, sysdig confirms the success of our filter by showing the one log message we wanted to isolate in this case)



Now we have **a small file that contains just the activity before and after that log message was written, without noise.** So let’s take a look. Note that I’m going to interject my comments to the trace output, to help the untrained eye spot the interesting parts.



<pre>$ sysdig -r request.scap
5104 23:40:09.103847117 3 httpd (28599) &lt; epoll_wait res=1 5105 23:40:09.103947771 3 httpd (28599) &gt; accept
5106 23:40:09.103960083 3 httpd (28599) &lt; accept fd=13(127.0.0.1:40016-&gt;127.0.0.1:80) tuple=127.0.0.1:40016-&gt;127.0.0.1:80 queuepct=0</pre>



<span style="color: #0000ff;"> This is where the connection is received by the apache server. The client IP address is 127.0.0.1.</span>



<pre>5107 23:40:09.104014005 3 httpd (28599) &gt; fcntl fd=13(127.0.0.1:40016-&gt;127.0.0.1:80) cmd=2(F_GETFD)
5108 23:40:09.104016357 3 httpd (28599) &lt; fcntl res=0(/dev/null)
5109 23:40:09.104016814 3 httpd (28599) &gt; fcntl fd=13(127.0.0.1:40016-&gt;127.0.0.1:80) cmd=3(F_SETFD)
5110 23:40:09.104017097 3 httpd (28599) &lt; fcntl res=0(/dev/null)
5111 23:40:09.104027550 3 httpd (28599) &gt; semop
5112 23:40:09.104759831 3 httpd (28599) &lt; semop 5113 23:40:09.104820227 3 httpd (28599) &gt; getsockname
5114 23:40:09.104826231 3 httpd (28599) &lt; getsockname 5115 23:40:09.105334639 3 httpd (28599) &gt; fcntl fd=13(127.0.0.1:40016-&gt;127.0.0.1:80) cmd=4(F_GETFL)
5116 23:40:09.105340241 3 httpd (28599) &lt; fcntl res=2(/opt/lampp/logs/error_log)
5117 23:40:09.105341064 3 httpd (28599) &gt; fcntl fd=13(127.0.0.1:40016-&gt;127.0.0.1:80) cmd=5(F_SETFL)
5118 23:40:09.105341663 3 httpd (28599) &lt; fcntl res=0(/dev/null)
5119 23:40:09.105899621 3 httpd (28599) &gt; switch next=0 pgft_maj=3 pgft_min=619 vm_size=442720 vm_rss=668 vm_swap=7004
5588 23:40:09.378390386 2 httpd (28599) &gt; read fd=13(127.0.0.1:40016-&gt;127.0.0.1:80) size=8000
5589 23:40:09.378418530 2 httpd (28599) &lt; read res=318 data=GET /userstats.php HTTP/1.1..Host: 127.0.0.1..User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:21.0) Gecko/20100101 Firefox/21.0..Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8..Accept-Language: en-US,en;q=0.5..Accept-Encoding: gzip, deflate..Connection: keep-alive..Cache-Control: max-age=0....</pre>



<span style="color: #0000ff;">The GET request arrives from the client. Among other things, we can see the URL, userstats.php. </span>



<pre>5590 23:40:09.378918776 2 httpd (28599) &gt; switch next=59720 pgft_maj=4 pgft_min=635 vm_size=442720 vm_rss=668 vm_swap=7004
5599 23:40:09.396427155 2 httpd (28599) &gt; switch next=0 pgft_maj=5 pgft_min=654 vm_size=442720 vm_rss=668 vm_swap=7004
5608 23:40:09.398102142 2 httpd (28599) &gt; stat
5609 23:40:09.398128234 2 httpd (28599) &lt; stat res=0 path=/opt/lampp/htdocs/userstats.php 5610 23:40:09.398233308 2 httpd (28599) &gt; open
5611 23:40:09.398249150 2 httpd (28599) &lt; open fd=-2(ENOENT) name=/opt/lampp/htdocs/.htaccess flags=1(O_RDONLY) mode=0 5634 23:40:09.403161359 2 httpd (28599) &gt; open
5635 23:40:09.403175465 2 httpd (28599) &lt; open fd=14(/opt/lampp/htdocs/userstats.php) name=/opt/lampp/htdocs/userstats.php flags=1(O_RDONLY) mode=0
5636 23:40:09.403245437 2 httpd (28599) &gt; fstat fd=14(/opt/lampp/htdocs/userstats.php)
5637 23:40:09.403250724 2 httpd (28599) &lt; fstat res=0 5638 23:40:09.403258650 2 httpd (28599) &gt; fstat fd=14(/opt/lampp/htdocs/userstats.php)
5639 23:40:09.403259412 2 httpd (28599) &lt; fstat res=0 5640 23:40:09.403273154 2 httpd (28599) &gt; fstat fd=14(/opt/lampp/htdocs/userstats.php)
5641 23:40:09.403274053 2 httpd (28599) &lt; fstat res=0 5642 23:40:09.403275792 2 httpd (28599) &gt; mmap addr=0 length=113 prot=1(PROT_READ) flags=1(MAP_SHARED) fd=14(/opt/lampp/htdocs/userstats.php) offset=0
5643 23:40:09.403294603 2 httpd (28599) &lt; mmap res=7F8250DC3000 vm_size=442724 vm_rss=2408 vm_swap=6336 5644 23:40:09.403542240 2 httpd (28599) &gt; munmap addr=7F8250DC3000 length=113
5645 23:40:09.403560506 2 httpd (28599) &lt; munmap res=0 vm_size=442720 vm_rss=2620 vm_swap=6296 5646 23:40:09.403572796 2 httpd (28599) &gt; close fd=14(/opt/lampp/htdocs/userstats.php)</pre>



<span style="color: #0000ff;">userstats.php is opened and loaded into memory.</span>



<pre>5647 23:40:09.403576281 2 httpd (28599) &lt; close res=0 5648 23:40:09.403620049 2 httpd (28599) &gt; getcwd
5649 23:40:09.403626400 2 httpd (28599) &lt; getcwd res=18 path=/opt/lampp/htdocs 5650 23:40:09.403633577 2 httpd (28599) &gt; lstat
5651 23:40:09.403641609 2 httpd (28599) &lt; lstat res=0 path=/opt/lampp/htdocs/./db_interface.php 5652 23:40:09.403648137 2 httpd (28599) &gt; open
5653 23:40:09.403656260 2 httpd (28599) &lt; open fd=14(/opt/lampp/htdocs/db_interface.php) name=/opt/lampp/htdocs/db_interface.php flags=1(O_RDONLY) mode=0
5654 23:40:09.403660357 2 httpd (28599) &gt; fstat fd=14(/opt/lampp/htdocs/db_interface.php)
5655 23:40:09.403661417 2 httpd (28599) &lt; fstat res=0 5656 23:40:09.403663079 2 httpd (28599) &gt; fstat fd=14(/opt/lampp/htdocs/db_interface.php)
5657 23:40:09.403663597 2 httpd (28599) &lt; fstat res=0 5658 23:40:09.403665748 2 httpd (28599) &gt; fstat fd=14(/opt/lampp/htdocs/db_interface.php)
5659 23:40:09.403666380 2 httpd (28599) &lt; fstat res=0 5660 23:40:09.403667145 2 httpd (28599) &gt; mmap addr=0 length=376 prot=1(PROT_READ) flags=1(MAP_SHARED) fd=14(/opt/lampp/htdocs/db_interface.php) offset=0
5661 23:40:09.403674583 2 httpd (28599) &lt; mmap res=7F8250DC3000 vm_size=442724 vm_rss=2620 vm_swap=6296 5662 23:40:09.404640219 2 httpd (28599) &gt; switch next=330 pgft_maj=6 pgft_min=1175 vm_size=442724 vm_rss=2620 vm_swap=6296
5663 23:40:09.405363736 2 httpd (28599) &gt; switch next=499(rngd) pgft_maj=6 pgft_min=1183 vm_size=442724 vm_rss=2620 vm_swap=6296
5664 23:40:09.405643687 2 httpd (28599) &gt; munmap addr=7F8250DC3000 length=376
5665 23:40:09.405661866 2 httpd (28599) &lt; munmap res=0 vm_size=442720 vm_rss=2868 vm_swap=6120 5666 23:40:09.405667140 2 httpd (28599) &gt; close fd=14(/opt/lampp/htdocs/db_interface.php)
5667 23:40:09.405669782 2 httpd (28599) &lt; close res=0</pre>



<span style="color: #0000ff;">A second php file, db_interface.php, is loaded into memory. It’s extremely likely that this file is imported by userstats.php. </span>



<pre>5668 23:40:09.406165738 2 httpd (28599) &gt; switch next=330 pgft_maj=6 pgft_min=1233 vm_size=442720 vm_rss=2868 vm_swap=6120
5669 23:40:09.406240303 2 httpd (28599) &gt; socket domain=10(AF_INET6) type=2 proto=0
5670 23:40:09.406372521 2 httpd (28599) &lt; socket fd=14()
5671 23:40:09.406386102 2 httpd (28599) &gt; close fd=14()
5672 23:40:09.406388724 2 httpd (28599) &lt; close res=0 5673 23:40:09.406444211 2 httpd (28599) &gt; socket domain=2(AF_INET) type=1 proto=0
5674 23:40:09.406545389 2 httpd (28599) &lt; socket fd=14()
5675 23:40:09.406547070 2 httpd (28599) &gt; fcntl fd=14() cmd=4(F_GETFL)
5676 23:40:09.406549281 2 httpd (28599) &lt; fcntl res=2(/opt/lampp/logs/error_log)
5677 23:40:09.406550034 2 httpd (28599) &gt; fcntl fd=14() cmd=5(F_SETFL)
5678 23:40:09.406550806 2 httpd (28599) &lt; fcntl res=0(/dev/null)
5679 23:40:09.406551722 2 httpd (28599) &gt; connect fd=14()
5680 23:40:09.406900932 2 httpd (28599) &lt; connect res=-115(EINPROGRESS) tuple=127.0.0.1:51525-&gt;127.0.0.1:3306</pre>



<span style="color: #0000ff;">A connection is established to port 3306 of localhost - this is the mysql port.</span>



<pre>5681 23:40:09.406912034 2 httpd (28599) &gt; poll fds=14:435 timeout=60000
5682 23:40:09.406918947 2 httpd (28599) &lt; poll res=1 fds=14:44 5683 23:40:09.406921328 2 httpd (28599) &gt; getsockopt
5684 23:40:09.406926197 2 httpd (28599) &lt; getsockopt 5685 23:40:09.406927738 2 httpd (28599) &gt; fcntl fd=14(127.0.0.1:51525-&gt;127.0.0.1:3306) cmd=5(F_SETFL)
5686 23:40:09.406929477 2 httpd (28599) &lt; fcntl res=0(/dev/null)
5687 23:40:09.406957933 2 httpd (28599) &gt; setsockopt
5688 23:40:09.406963959 2 httpd (28599) &lt; setsockopt 5689 23:40:09.406977520 2 httpd (28599) &gt; poll fds=14:431 timeout=1471228928
5695 23:40:09.407004946 2 httpd (28599) &gt; switch next=0 pgft_maj=6 pgft_min=1259 vm_size=442720 vm_rss=2868 vm_swap=6120
5718 23:40:09.407726899 3 httpd (28599) &lt; poll res=1 fds=14:41 5719 23:40:09.407772280 3 httpd (28599) &gt; recvfrom fd=14(127.0.0.1:51525-&gt;127.0.0.1:3306) size=4
5720 23:40:09.407784644 3 httpd (28599) &lt; recvfrom res=4 data=J... tuple=NULL 5721 23:40:09.407796522 3 httpd (28599) &gt; poll fds=14:431 timeout=1471228928
5722 23:40:09.407801177 3 httpd (28599) &lt; poll res=1 fds=14:41 5723 23:40:09.407802020 3 httpd (28599) &gt; recvfrom fd=14(127.0.0.1:51525-&gt;127.0.0.1:3306) size=78
5724 23:40:09.407806960 3 httpd (28599) &lt; recvfrom res=74 data=.5.5.31.....,N(wD+Ku...................0)}]8@uQrkCM.mysql_native_password. tuple=NULL 5725 23:40:09.407878378 3 httpd (28599) &gt; sendto fd=14(127.0.0.1:51525-&gt;127.0.0.1:3306) size=85 tuple=NULL
5726 23:40:09.407994004 3 httpd (28599) &lt; sendto res=85 data=Q...................................root..5.*......S....8.u]=..mysql_native_password.</pre>



<span style="color: #0000ff;">An authentication request is sent to mysql. User is root. </span>



<pre>5727 23:40:09.408007187 3 httpd (28599) &gt; poll fds=14:431 timeout=1471228928
5728 23:40:09.408024570 3 httpd (28599) &gt; switch next=0 pgft_maj=6 pgft_min=1270 vm_size=442720 vm_rss=3120 vm_swap=6100
5735 23:40:09.408397503 3 httpd (28599) &lt; poll res=1 fds=14:41 5736 23:40:09.408401449 3 httpd (28599) &gt; recvfrom fd=14(127.0.0.1:51525-&gt;127.0.0.1:3306) size=4
5737 23:40:09.408408712 3 httpd (28599) &lt; recvfrom res=4 data=H... tuple=NULL 5738 23:40:09.408415661 3 httpd (28599) &gt; poll fds=14:431 timeout=1471228928
5739 23:40:09.408418497 3 httpd (28599) &lt; poll res=1 fds=14:41 5740 23:40:09.408419273 3 httpd (28599) &gt; recvfrom fd=14(127.0.0.1:51525-&gt;127.0.0.1:3306) size=78
5742 23:40:09.408421912 3 httpd (28599) &lt; recvfrom res=72 data=...#28000Access denied for user 'root'@'localhost' (using password: YES) tuple=NULL</pre>



<span style="color: #0000ff;">mysql rejects the credentials! </span>



<pre>5743 23:40:09.408507305 3 httpd (28599) &gt; close fd=14(127.0.0.1:51525-&gt;127.0.0.1:3306)
5744 23:40:09.408509917 3 httpd (28599) &lt; close res=0</pre>



<span style="color: #0000ff;">The connection to mysql gets closed.</span>



<pre>5745 23:40:09.408621532 3 httpd (28599) &gt; open
5746 23:40:09.408647914 3 httpd (28599) &lt; open fd=14(/opt/lampp/logs/php_error_log) name=/opt/lampp/logs/php_error_log flags=14(O_APPEND|O_CREAT|O_WRONLY) mode=0
5747 23:40:09.408938827 3 httpd (28599) &gt; write fd=14(/opt/lampp/logs/php_error_log) size=52
5748 23:40:09.408991270 3 httpd (28599) &lt; write res=52 data=[31-Jul-2014 05:40:09] Database error. 5749 23:40:09.408994303 3 httpd (28599) &gt; close fd=14(/opt/lampp/logs/php_error_log)
5750 23:40:09.408996256 3 httpd (28599) &lt; close res=0</pre>



<span style="color: #0000ff;">This is where the log message is written. </span>



<pre>5836 23:40:09.414189905 0 httpd (28599) &gt; read fd=13(127.0.0.1:40016-&gt;127.0.0.1:80) size=8000
5837 23:40:09.414202406 0 httpd (28599) &lt; read res=-11(EAGAIN) data= 5838 23:40:09.414210168 0 httpd (28599) &gt; writev fd=13(127.0.0.1:40016-&gt;127.0.0.1:80) size=353
5839 23:40:09.414279449 0 httpd (28599) &lt; writev res=353 data=HTTP/1.1 200 OK..Date: Thu, 31 Jul 2014 03:40:09 GMT..Server: Apache/2.4.4 (Unix) OpenSSL/1.0.1e PHP/5.4.16 mod_perl/2.0.8-dev Perl/v5.16.3..X-Powered-By: PHP/5.4.16..Content-Length: 83..Keep-Alive: timeout=5, max=100..Connection: Keep-Alive..Content-Type: text/html.... 5840 23:40:09.414428358 0 httpd (28599) &gt; write fd=9(/opt/lampp/logs/access_log) size=80</pre>



<span style="color: #0000ff;">The response reporting the failure is sent to the client browser.</span>



<pre>5842 23:40:09.414470191 0 httpd (28599) &lt; write res=80 data=127.0.0.1 - - [30/Jul/2014:23:40:09 -0400] "GET /userstats.php HTTP/1.1" 200 83. 5843 23:40:09.414475321 0 httpd (28599) &gt; times
5845 23:40:09.414478852 0 httpd (28599) &lt; times 5846 23:40:09.414502519 0 httpd (28599) &gt; poll fds=13:41 timeout=5000
5847 23:40:09.414523199 0 httpd (28599) &gt; switch next=0 pgft_maj=6 pgft_min=1500 vm_size=442720 vm_rss=3684 vm_swap=5840</pre>



<span style="color: #0000ff;">Apache is done and is going to sleep until the next request arrives.</span>



So, to summarize: a script called db_interface.php, imported by userstats.php, is trying to connect to mysql, but is using the wrong credentials. mysql refuses the connection and the request fails.



The solution: go check the password!



## Conclusion



This blog post only scratches the surface of what sysdig can do with your system logs, but hopefully it’s a good start to get a basic idea and start hacking.



The chisels that I presented here focus on (1) easy system-wide log collection and (2) the ability to dive deeper by correlating log entries with relevant system information. The more I work on sysdig, however, the more I’m amazed at how much useful information is buried in every system. The trick is to bubble it up in a way that is easy and productive. So stay tuned, because we are working on several more powerful features here.



And, as usual, we’d love to hear what you think. If you have any thoughts or suggestions, please let us know in the comments or on twitter: [@sysdig][4].

Happy digging!

 [1]: http://www.sysdig.org/
 [2]: https://github.com/draios/sysdig/wiki
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/08/Screen-Shot-2014-08-04-at-1.30.57-PM.png
 [4]: https://twitter.com/sysdig