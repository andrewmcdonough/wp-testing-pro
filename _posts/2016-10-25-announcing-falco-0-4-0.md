---
ID: 3268
post_title: Announcing Falco 0.4.0
author: Mark Stemm
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/announcing-falco-0-4-0/
published: true
post_date: 2016-10-25 13:48:35
---
## Falco Release 0.4.0

Yesterday we released Falco 0.4.0. It's been a couple of months since 0.3.0, but there are lots of new features!

The biggest change is greatly improved visibility into container and orchestration information when matching events against the set of falco rules. For containers, you can take advantage of new filterchecks <tt>%container.privileged</tt> and <tt>%container.mount.*</tt> that allow for detecting events that occur within privileged containers or containers that have specific mounts.

We've also added some rules to the default ruleset that take advantage of these features: *File Open by Privileged Container* and *Sensitive Mount by Container*. Here are the new rules:

<script src="https://gist.github.com/a95966413162e990bfee8a9fdea86067.js"></script> <noscript>
  <pre><code>
File: Falco 0.4.0 Sample Rules
------------------------------


</code></pre>
</noscript>

The first rule detects attempts to open files by processes running in privileged containers. The second rule detects attempts to open files by processes running in containers that have mounted a "sensitive" path (currently <tt>/proc</tt>) from the host. For both rules, there's a list of known good images in the macro <tt>trusted_containers</tt>.

### Kubernetes and Marathon Support

For orchestration, we've brought over the ability to communicate with K8s/Marathon servers to decorate events with appropriate pod/deployment/framework/etc information, which you can then use in rules or output strings. In many rules, <tt>%container.info</tt> is used in the output string to provide either container-level, k8s-level, or mesos-level information, depending on the context in which the event occurred and how falco was started. For more information, see our <a href="https://github.com/draios/falco/wiki/Falco%20Formatting%20for%20Containers%20and%20Orchestration" target="_blank" rel="noopener">wiki page</a> on the topic.

We've also added a new test program <tt>event_generator</tt> and corresponding docker image <a href="https://hub.docker.com/r/sysdig/falco-event-generator/" target="_blank" rel="noopener">sysdig/falco-event-generator</a> that allows you to see falco in action. Simply running <tt>docker run -it sysdig/falco-event-generator</tt> downloads and runs the test program, which does all kinds of evil things like overwriting files below <tt>/bin</tt>, reading files below <tt>/etc</tt>, spawning shells in containers, opening network connections from non-network-capable programs like <tt>ls</tt>, etc. Run falco alongside the event generator and you'll see it detect all of these activities.

### Addition features

We've also added lots of new smaller features, including:

*   A <tt>glob</tt> operator that allows matching against pathnames using wildcards. 
*   A <tt>pmatch</tt> operator that allows testing a subject pathname (i.e. <tt>/usr/local/bin/perl</tt>) against a set of target pathnames (i.e. <tt>/lib/x86_64-linux-gnu/</tt>,<tt>/var/www/</tt>,<tt>/etc/</tt>, ...), to see if the subject is a prefix of any of the targets. 
*   Verbose output with <tt>-v</tt> now includes stats on the number of events processed and dropped. 
*   The ability to write trace files with <tt>-w</tt>. This can be useful to write a trace file in parallel with live event monitoring so you can reproduce it later.  For the full set of changes in this release, you can always look at the 

[changelog at github][1].  
  
The release is available via the usual channels–rpm/debian packages, [docker hub][2], [github][3].  
  
Finally, if you want the complete story on Falco, [head over to the website][4] and read all about it.  
  
Let us know if you have any issues, and enjoy!

 [1]: https://github.com/draios/falco/blob/master/CHANGELOG.md
 [2]: https://hub.docker.com/r/sysdig/falco
 [3]: https://github.com/draios/falco
 [4]: http://www.sysdig.org/falco/