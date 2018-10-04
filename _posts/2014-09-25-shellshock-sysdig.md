---
ID: 438
post_title: >
  Easy, realtime, system-wide Shellshock
  monitoring
author: Loris Degioanni
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/shellshock-sysdig/
published: true
post_date: 2014-09-25 17:30:08
---
The world hasn’t had time to recover from the chaos generated from the Heartbleed OpenSSL bug, and we already have another massive vulnerability jeopardizing the whole internet. The CVE-2014-6271, also known as “[Shellshock][1]”, targets bash, the most popular unix shell in the world, with a malicious environment variable that allows arbitrary execution. 
## Detecting Shellshock Intrusions Since the bug is still very fresh and the patches are still under validation, it’s critical to detect and log attack attempts to your systems. The typical way to do it is through network intrusion detection systems like snort, but this approach is less than ideal: 

*   It works by looking at the network payloads containing the attack signature, which can generate false positives, because you can’t know if the payload actually generated a bash execution.
*   It only sees network attacks coming from well known protocols, while Shellshock can come from a number of vectors, several of which have probably not been discovered yet. This means that many successful attacks could be missed. Fortunately, 

[sysdig][2] can help. 
## The Shellshock Chisel Using sysdig to capture all the bash executions is trivial*, but in order to make things even easier, we spent a couple hours today putting together a new sysdig release that contains a new chisel called 

*shellshock_detect*. The chisel captures all the bash executions for which the environment variables match the Shellshock signature, and for each of them it prints 
*   The time
*   The victim process’ name and pid (ie. the process that has been attacked with the malicious payload and that will execute bash)
*   The injected function (i.e. what bash is going to execute)

## tl;dr: Taking Direct Action Want to see what happens on your machines? Just 

[update sysdig][3], and then run: 
    sysdig -c shellshock_detect If somebody is attacking you, you will start seeing an output similar to this: 

<pre>TIME                  PROCNAME              PID     FUNCTION
13:51:18.779785087    apache2               2746    () { test;};echo "Content-type: text/plain"; echo; echo; /bin/cat /etc/passwd
13:52:31.459230424    dhclient              2896    () { :;} ; echo busted</pre>

[See here][4] for the source code, if you're curious about how the chisel works. We're constantly working to come up with new and interesting uses for sysdig - if you have any ideas, we'd love to hear them. Comment below or tweet [@sysdig][5]. Stay safe out there.   **For example, try:* 
<pre><em><code>sysdig "proc.name contains bash and evt.type=execve and evt.dir = &lt;"
</code></em></pre>

<script src="resource://ember-inspector-at-emberjs-dot-com/ember-inspector/data/in-page-script.js"></script>

 [1]: http://www.troyhunt.com/2014/09/everything-you-need-to-know-about.html
 [2]: http://www.sysdig.org/
 [3]: https://github.com/draios/sysdig/wiki/Sysdig%20Update%20and%20Uninstall
 [4]: https://github.com/draios/sysdig/blob/master/userspace/sysdig/chisels/shellshock_detect.lua
 [5]: https://twitter.com/sysdig