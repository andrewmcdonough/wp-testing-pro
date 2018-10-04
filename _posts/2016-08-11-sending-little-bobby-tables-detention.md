---
ID: 2931
post_title: Sending little bobby tables to detention
author: Mark Stemm
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sending-little-bobby-tables-detention/
published: true
post_date: 2016-08-11 17:33:44
---
Little Bobby Tables shows us why it's a good idea to sanitize your database inputs to avoid SQL injection attacks:  
  




[][1][<img alt="Little Bobby Tables" class="alignnone" height="536" src="/wp-content/uploads/2016/08/bobby-tables.png" width="750" />][1] 



  
In case you're not familiar with the concept of SQL injection attacks, here's a quick summary:



*   Poorly written software uses a combination of a sql statement fragment like <tt>select salary from employees where id=</tt> and user provided data, without sanitizing the inputs or properly binding the configurable part of the statement to an input value.
*   An attacker executes the query with a second sql statement tacked on to the user provided data. For example, the attacker could provide <tt>10; drop database employees;</tt> as the "argument" to the select query. The full statement that would be executed is <tt>select salary from employees where id=10; drop database employees;</tt>.



This is especially dangerous for some database types that allow executing [arbitrary shells][2] or [reading][3]/[writing][4] arbitrary files. Now, by exploiting a security hole in your database an attacker has full access to your system.



Like Little Bobby's mom says, it would be great if all our software properly sanitized their database inputs to stop SQL injection attacks. In addition to sanitizing inputs, there are plenty of other ways to limit the severity of attacks: limiting database privileges, removing unused stored procedures, disabling unneeded functionality, etc. Unfortunately, there's a lot of code out there issuing queries against databases, and a lot of database configurations, and it's very difficult to ensure all of them have been audited and fixed. All it takes is one vulnerability for an attacker to start executing arbitrary sql against your database.



So if you assume you’re not going to fix all the bad code out there, how do you protect your database against malicious intent? Another way to provide security is to *detect* attacks by looking for the things an attacker does after they've exploited a vulnerability:



*   Reading sensitive files
*   Deleting or replacing system files to change a server's configuration
*   Spawn a program like a shell
*   Installing new scripts below a web server root that act as reverse shells and open network connections



Detecting these types of behaviors is exactly why we wrote sysdig falco.



## How Sysdig Falco Can Help



Sysdig falco is an open source, behavioral activity monitor designed to detect anomalous activity in applications. Falco is powered by a set of rules that identify suspect behavior based on the system calls that applications perform. Falco works in containerized, virtualized, and bare metal linux deployments. Here's an example of a falco rule:  
  




<pre>- macro: bin_dir
  condition: fd.directory in (/bin, /sbin, /usr/bin, /usr/sbin)

- macro: open_write
  condition: (evt.type=open or evt.type=openat) and evt.is_open_write=true and fd.typechar='f'

- list: package_mgmt_binaries
  items: [dpkg, dpkg-preconfigu, rpm, rpmkey, yum, frontend]

- rule: write_binary_dir
  desc: an attempt to write to any file below a set of binary directories
  condition: bin_dir and evt.dir = &lt; and open_write and not package_mgmt_procs
  output: "File below a known binary directory opened for writing (user=%user.name command=%proc.cmdline file=%fd.name)"
  priority: WARNING
</pre>



  
The rule <tt>write_binary_dir</tt> detects unexpected writes to files below known binary directories. The <tt>condition</tt> field is a sysdig [filtering expression][5] that examines all system calls and identifies those system calls that represent suspect activity. This is not just limited to system calls and their arguments. Sysdig libraries build up low-level system call information and system state into *events* that provide a rich view into the state of an application. Sysdig also has support for containers and orchestration frameworks like mesos, kubernetes, and docker swarm.



When an event matches the rule's condition, falco sends a notification using the rule's <tt>output</tt> and <tt>priority</tt> fields.



This example also shows some additional features that falco's ruleset provides on top of sysdig filtering expressions. Falco's ruleset can also contain *macros* and *lists*--reusable snippets that can be included in other rules to aid in legibility.



Now let's show how falco's rules can be used to detect an attacker's malicious activity after performing a SQL injection attack. Most of these rules are taken verbatim from the 0.3.0 [rule set][6].



## Detecting Malicious Activity



### Reading Sensitive Files



One thing an attacker might do is try to read sensitive files like <tt>/etc/shadow</tt> to obtain things like user/password lists. Here are falco rules that detect this behavior:  
  




<pre>- macro: sensitive_files
  condition: fd.name startswith /etc and (fd.name in (/etc/shadow, /etc/sudoers, /etc/pam.conf) or fd.directory in (/etc/sudoers.d, /etc/pam.d))

- macro: server_procs
  condition: proc.name in (http_server_binaries, db_server_binaries, docker_binaries, sshd)

- macro: proc_is_new
  condition: proc.duration &lt;= 5000000000

- rule: read_sensitive_file_untrusted
  desc: an attempt to read any sensitive file (e.g. files containing user/password/authentication information). Exceptions are made for known trusted programs.
  condition: sensitive_files and open_read and not proc.name in (user_mgmt_binaries, userexec_binaries, package_mgmt_binaries, cron_binaries, iptables, ps, lsb_release, check-new-relea, dumpe2fs, accounts-daemon, bash, sshd) and not proc.cmdline contains /usr/bin/mandb
  output: "Sensitive file opened for reading by non-trusted program (user=%user.name command=%proc.cmdline file=%fd.name)"
  priority: WARNING

- rule: read_sensitive_file_trusted_after_startup
  desc: an attempt to read any sensitive file (e.g. files containing user/password/authentication information) by a trusted program after startup. Trusted programs might read these files at startup to load initial state, but not afterwards.
  condition: sensitive_files and open_read and server_procs and not proc_is_new and proc.name!="sshd"
  output: "Sensitive file opened for reading by trusted program after startup (user=%user.name command=%proc.cmdline file=%fd.name)"
  priority: WARNING
</pre>



  
There are actually two rule variants. The first detects any attempt to access a sensitive file, with exclusions for known trusted programs. The second detects an attempt by a program to access a sensitive file, *after* a configurable initial period (the <tt>proc_is_new/proc.duration <= 5000000000</tt> macro). This is useful for some programs, including databases, that read files at startup while they load user/account information, but don't tend to access them once they are up and running.



### Deleting/Replacing System Files



An attacker might also try to overwrite system files like files below <tt>/etc/</tt>, etc, to change a server's configuration. Here's how we can detect that in falco:  
  




<pre>- macro: write_etc_common
  condition: &gt;
    etc_dir and evt.dir = &lt; and open_write
    and not proc.name in (shadowutils_binaries, sysdigcloud_binaries, package_mgmt_binaries, ssl_mgmt_binaries, dhcp_binaries, ldconfig.real)
    and not proc.pname in (sysdigcloud_binaries)
    and not fd.directory in (/etc/cassandra, /etc/ssl/certs/java)

- rule: write_etc
  desc: an attempt to write to any file below /etc, not in a pipe installer session
  condition: write_etc_common and not proc.sname=fbash
  output: "File below /etc opened for writing (user=%user.name command=%proc.cmdline file=%fd.name)"
  priority: WARNING
</pre>



  
The macro <tt>write_etc_common</tt> does most of the work, looking for the attempt to open a file below <tt>/etc/</tt> by a non-trusted program. <tt>write_etc</tt> uses that macro along with <tt>proc.name=fbash</tt>, which is a way to identify installation sessions. See our [blog post][7] for a discussion of pipe installers and installation sessions.



### Spawning Shells



An attacker might also try to spawn a shell after exploiting a vulnerability. Here's how falco detects this behavior.  
  




<pre>- macro: spawned_process
  condition: evt.type = execve and evt.dir=&lt;

- rule: run_shell_untrusted
  desc: an attempt to spawn a shell by a non-shell program. Exceptions are made for trusted binaries.
  condition: spawned_process and not container and proc.name = bash and proc.pname exists and not proc.pname in (cron_binaries, bash, sshd, sudo, docker, su, tmux, screen, emacs, systemd, login, flock, fbash, nginx, monit, supervisord, dragent)
  output: "Shell spawned by untrusted binary (user=%user.name shell=%proc.name parent=%proc.pname cmdline=%proc.cmdline)"
  priority: WARNING
</pre>



  
The rule looks for any attempt to run a program <tt>bash</tt> when the parent program is not in a set of programs known to spawn shells. Note that database programs are not in the list, so if someone were to exploit a loophole to invoke a shell, falco would detect it.



### Opening Network Connections



An attacker might also install software that opens a network connection to "phone home" to an attacker. They could do this in a few ways:



*   They might overwrite a common binary program like <tt>split</tt>, <tt>stat</tt> etc, with a completely different program that acts as a reverse shell. This makes the reverse shell harder to detect.
*   They might install a php/perl script below a web server root directory and then invoke the script using the web server's scripting module (<tt>mod_php</tt>, <tt>mod_perl</tt>, etc.).



Here are falco rules that can detect this behavior:  
  




<pre>- macro: inbound
  condition: ((evt.type=listen and evt.dir=&gt;) or (evt.type=accept and evt.dir=&lt;))

- macro: outbound
  condition: evt.type=connect and evt.dir=&lt; and (fd.typechar=4 or fd.typechar=6)

- macro: system_procs
  condition: proc.name in (coreutils_binaries, user_mgmt_binaries)

- rule: system_procs_network_activity
  desc: any network activity performed by system binaries that are not expected to send or receive any network traffic
  condition: (fd.sockfamily = ip and system_procs) and (inbound or outbound)
  output: "Known system binary sent/received network traffic (user=%user.name command=%proc.cmdline connection=%fd.name)"
  priority: WARNING

- rule: parent_apache_network_activity
  desc: any network activity performed by a program that is a child of apache. Apache might run php/perl scripts via mod_php/mod_perl, but the scripts should not make network connections.
  condition: fd.sockfamily = ip and (inbound or outbound) and proc.aname=apache
  output: "Known system binary sent/received network traffic (user=%user.name command=%proc.cmdline connection=%fd.name)"
  priority: WARNING
</pre>



  
The first rule detects any attempt by a coreutils/user management binary to open an inbound or outbound network connection. These programs are not expected to perform any network activity, so if they did, it's a sign that they have been tampered with.



The second rule looks for abuses of <tt>mod_php</tt>/<tt>mod_perl</tt>, detecting any network connection by a program who has apache somewhere in its ancestor tree (the <tt>proc.aname=apache</tt> part).



## Conclusion



We've shown that SQL injection attacks can allow an attacker to gain privileged access to a target system. In the worst case this could include the ability to read/write arbitrary files and spawn new programs.



We've shown how sysdig falco can be used to provide an additional layer of protection against these attacks by detecting the activities the attacker performs after exploiting the vulnerability.



Hopefully this gets you excited about sysdig falco. Now go [download][8] falco and start using it for yourself!



By the way, we’re always looking for new contributions to the rule set. Please [add any rules][6] that would be helpful for the community!

 [1]: /wp-content/uploads/2016/08/bobby-tables.png
 [2]: https://msdn.microsoft.com/en-us/library/ms175046.aspx
 [3]: http://dev.mysql.com/doc/refman/5.7/en/load-data.html
 [4]: http://dev.mysql.com/doc/refman/5.7/en/select-into.html
 [5]: https://github.com/draios/sysdig/wiki/Sysdig-User-Guide#filtering
 [6]: https://github.com/draios/falco/blob/dev/rules/falco_rules.yaml
 [7]: https://sysdigrp2rs.wpengine.com/blog/friends-dont-let-friends-curl-bash/
 [8]: https://github.com/draios/falco/wiki/How%20to%20Install%20Falco%20using%20Containers