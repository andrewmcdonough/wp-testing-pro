---
ID: 2615
post_title: 'Introducing Falco: Open source, behavioral security from Sysdig'
author: Henri Dubois-Ferriere
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-falco/
published: true
post_date: 2016-05-16 23:00:04
---
*There are a million ways a burglar can break into your home, but once they do they’re going to steal your jewelry. *

  
Today, we’re releasing [sysdig falco][1], a behavioral activity monitoring agent that is open source and comes with native support for containers. Falco lets you define highly granular rules to check for activities involving file and network activity, process execution, IPC, and much more, using a flexible syntax. Falco will notify you when these rules are violated.

You can think about falco as a mix between snort, ossec and strace. 

We created falco based on the premise that there is an ongoing shift away from signature-based security monitoring and towards behavioral security monitoring. Signature-based approaches, which must list each possible exploit, vulnerability, or attack in some way (network packets, malware signatures, …), are engaged in a never-ending game of catch up with the constant stream of new threats. 

Behavioral approaches, in contrast, look at what is happening on a system. In other words they “only” need to detect the things that an attacker does once they have access to a system, rather than all the ways an attacker can gain access to a system.



This is a large-scale trend. Within the behavioral space, the fundamental novelty of falco lies in its instrumentation point: the operating system call interface. Why does that matter? Simply because most host activity with potential security implications happens via system calls: processes are created through syscalls; file and network IO happen through syscalls, signaling is done through syscalls... the list goes on...



Let’s take a couple of examples.



## How Falco Compares to Existing Approaches



*   **File integrity monitoring:** One way to do file integrity monitoring is to periodically scan all files of interest, compute their checksums, and compare with the checksums of the previous phase. The challenge with this kind of approach is that it is costly to scan all files, so one typically runs it every few hours. With falco, we can just watch for any OS activity that is writing to a file of interest, and be alerted in real-time.
*   **Network monitoring:** Network-based IDS systems have been around for a long time. But as workloads moved to VMs and containers, it has become increasingly hard to reconcile network traffic with application activity. Furthermore, network-based IDS is inextricably tied to signatures, since you can only observe a small slice of system behavior from network traffic. With falco, we can see I/O “from the inside”, with an immediate correlation between applications and traffic.
*   **Integrated OS facilities:** Linux has multiple security modules, notably including SELinux and AppArmor. These are extremely advanced access control systems with sophisticated policies and concepts. As a result, understanding and configuring them is a rather complex undertaking. Falco is far simpler to understand and configure. If you’ve had a hard time with complex security products in the past, don’t be discouraged, and try out falco! (Note that one major difference is that falco is currently detection-only, while AppArmor and SELinux can detect and/or enforce. But this is orthogonal to the configuration complexity).



Enough talk. Let’s get into specifics on how falco works and what kind of things it can detect.



## How Falco works



Falco is a long-running server agent. In containerized environments, it can install as a container which monitors the host itself and all containers running on it. Of course, it can also be installed as a regular host package.



Once deployed, falco taps into the stream of system call events, checking each event against the list of rules in its configuration file.



Falco is built on top of the core software that powers [sysdig’s open source troubleshooting tool][2]. Specifically, it uses the sysdig kernel module for syscall interception and sysdig user libraries for state tracking and event decoding. Both are deployed and running in hundreds of thousands of machines today. This means very good stability despite the initial “0.1.0” version. On the other hand performance is still a work in progress. On busy hosts or with large rule sets, you may see the current version of falco using high CPU. Expect big improvements in coming releases.



Behaviors and activities of interest (such as the ones above) are expressed as rules using a simple filter language that is derived from [sysdig’s filters][3]. Each rule has an associated output template specifying the message to be output if a matching event occurs. Note that falco does not attempt to do collection, alerting, reporting, or remediation. Following the “do one thing” Unix philosophy, Falco is an edge agent that is intended to integrate with whichever systems you already use for reporting and monitoring.



## Falco Detection Examples



### Shell in a container



In production, you probably don’t expect a shell to be running inside your container. You can detect that with the following rule:



<pre>- <span style="color: #008000;">rule:</span> <span style="color: #0000ff;">shell_in_container</span>
  <span style="color: #008000;">desc:</span> <span style="color: #0000ff;">a shell running in a container</span>
  <span style="color: #008000;">condition:</span> <span style="color: #0000ff;">container.id != host and proc.name = bash</span>
  <span style="color: #008000;">output:</span> <span style="color: #0000ff;">“Shell running in container (user=%user.name container_id=%container.id container_name=%container.name shell=%proc.name parent=%proc.pname)”</span>
  <span style="color: #008000;">priority:</span> <span style="color: #0000ff;">WARNING
</span></pre>



The key part here is the condition field - it is what defines the events of interest for this rule. In this condition, the first clause checks that the system call happened in a container, and the second checks the process name. If you’re familiar with sysdig, you’ll immediately have recognized the sysdig syntax.



The other important field is the output: it contains the template string to be instantiated and output if a matching event occurs.



(A slightly more advanced version of this rule is present in the Falco rule set; it additionally excludes container entrypoints).



It’s worth emphasizing here that this detection happens across all your containers on the host where you’ve installed falco - just run the falco container, and it will monitor all of that host’s containers for you, out of the box, including the ones that start later on.



### Compromised server process



You know those painful SQL injection attacks that keep compromising your servers? Or web server vulnerabilities, general code execution attacks, php exploits, and others? All of these are hard to detect with traditional network-based pattern matching, and they require you to constantly update your signature base.



But at the end of the day most of these attacks typically result in unexpected server behavior such as spawning a process, or writing unexpected files. Falco can detect such behaviors reliably with just a few simple rules.



For example, you could write a rule that detects when a process is started and analyzes its properties such as executable name and parent name:



<pre>- <span style="color: #339966;">rule:</span> <span style="color: #0000ff;">mysqld_spawn_process</span>
  <span style="color: #008000;">desc:</span> <span style="color: #0000ff;">mysqld spawning a new process after startup. This shouldn't occur and is a follow on from some SQL injection attacks.</span>
  <span style="color: #008000;">condition:</span> <span style="color: #0000ff;">spawn_process and proc.name = mysqld and not proc_is_new</span>
  <span style="color: #008000;">output:</span> <span style="color: #0000ff;">“mysqld spawned new process after startup (user=%user.name command=%proc.cmdline file=%fd.name)”</span>
  <span style="color: #008000;">priority:</span> <span style="color: #0000ff;">WARNING
</span></pre>



This rule uses a simple sysdig-like filter on proc.name, as well as two terms: spawn_process and proc_is_new. What are these terms? They are references to macros. Macros are simply definitions that can be re-used inside rules and other macros, providing a way to factor out and name common patterns. Here’s what the macro definitions for these two terms look like:



<pre>- <span style="color: #008000;">macro:</span> <span style="color: #0000ff;">spawn_process</span>
  <span style="color: #008000;">condition:</span> <span style="color: #0000ff;">syscall.type = execve
</span></pre>



<pre>- <span style="color: #008000;">macro:</span> <span style="color: #0000ff;">proc_is_new</span>
  <span style="color: #008000;">condition:</span> <span style="color: #0000ff;">proc.duration &lt;= 5000000000
</span></pre>



The first macro is simply a check for the execve system call, which executes a program. The second uses the time since a process was started, with a threshold of five seconds, to determine if a process is “new”.



### System binary makes outbound connection



If a standard system binary like ls or ps makes an outbound TCP connection, something is wrong - a likely explanation being that the host has been rootkit’ed. While you’d preferably want to detect a rootkit installation when it happens (possibly using the kinds of rules described above), it remains important to defend in depth and detect behaviors that can happen after an attack is under way.



In this case, for example, we are detecting an outbound connection with a rule that considers the ‘connect’ system call, and looks at the process or executable name to check if it’s a standard system binary:



<pre>- <span style="color: #008000;">macro:</span> <span style="color: #0000ff;">open_connection</span>
  <span style="color: #008000;">condition:</span> <span style="color: #0000ff;">syscall.type=connect and evt.dir=&lt; and fd.sockfamily =ip</span>

- <span style="color: #008000;">rule:</span> <span style="color: #0000ff;">system_binaries_network_activity</span>
  <span style="color: #008000;">desc:</span> <span style="color: #0000ff;">any network connection initiated by system binaries that are not expected to send or receive any network traffic</span>
  <span style="color: #008000;">condition:</span> <span style="color: #0000ff;">open_connection and proc.name in (ls, ps, mkdir, … )</span> 
  <span style="color: #008000;">output:</span> <span style="color: #0000ff;">Known system binary made network connection (user=%user.name command=%proc.cmdline connection=%fd.name)</span>
  <span style="color: #008000;">priority:</span> <span style="color: #0000ff;">WARNING
</span></pre>



(Note that the actual rule that ships with falco enumerates all system binaries from various packages, not just the three in the condition above).



Let’s stop for a second and think about how we’d do this with a network-based approach (firewall or IDS). Looking at packets there’s no way to associate TCP connections with host processes. And since the compromised binary may access a perfectly legitimate host (e.g. downloading a tarball from S3), we can’t even resort to destination whitelisting as a general solution. Inspecting network activity in the context of processes, and therefore understanding if a connection on a well known port is legitimate tends to be hard with traditional approaches, but is very easy when sitting on top of the sysdig engine.



## Developing rules



We’ve provided a starting rule set that runs out of the box with falco, but that ruleset only scratches the surface of what’s possible, and definitely doesn’t cover every possible behavior of interest. We’d love to see PRs with new rules, and if you need help writing them, please stop by <google groups="" address=""> and we’ll be happy to assist.</google>



### Back-testing for rule development and testing



One of the really cool, practical features that comes with falco is the ability to back-test rules on historical capture files. If you’ve ever configured a security monitoring system, you might have gone through repeated iterations of a workflow like:



1.  Tweak configuration
2.  Manually reproduce “security incident”
3.  Verify that incident was caught (if not, goto 1)



With falco, there’s a better way, thanks to its ability to read regular sysdig capture files. You reproduce the security incident just one time while capturing events with sysdig, and then you can replay the capture file each time you run falco during rule development. Way, way easier. And you could eventually automate this, testing the output of falco on capture files e.g. each time you make a configuration change or system upgrade!



## Conclusion



We think falco fills an under-served sweet spot in behavioral security monitoring: an open-source agent that is lightweight, easy to configure, and works seamlessly with containers.



You can install falco from rpm/deb packages, or as container - this way, you instantly monitor activity on all of your containers, without making a single change to any of their images.



This is just the start, we’re really excited about the next steps for this project. Most of all we want you to [try this][4] and let us know what you think!

 [1]: http://www.sysdig.org/falco/
 [2]: http://www.sysdig.org/
 [3]: https://github.com/draios/sysdig/wiki/Sysdig-User-Guide#filtering
 [4]: https://github.com/draios/falco