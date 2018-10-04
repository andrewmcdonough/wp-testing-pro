---
ID: 2916
post_title: Announcing Falco 0.3.0
author: Mark Stemm
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/announcing-falco-0-3-0/
published: true
post_date: 2016-08-08 11:10:28
---
On Friday we released Falco 0.3.0. The biggest change in this release is significantly reduced cpu usage, involving changes in falco as well as the underlying sysdig libraries that falco uses:  
*   Reordering a rule condition's operators to put likely-to-fail operators at the beginning and expensive operators at the end. This allows rules to shortcut early when they don't match.
*   Adding the ability to perform <tt>x in (a, b, c, ...)</tt> as a single set membership test instead of individual comparisons between x=a, x=b, etc.
*   Avoid unnecessary string manipulations.
*   Using <tt>startswith</tt> as a string comparison operator when possible.
*   Use <tt>is_open_read</tt>/<tt>is_open_write</tt> when possible instead of searching through open flags.
*   Group rules by event type, which allows for an initial filter using event type before going through each rule's condition.All of these changes result in dramatically reduced CPU usage. Here are some comparisons between 0.2.0 and 0.3.0 for the following workloads:

  
*   [Phoronix][1]'s <tt>pts/apache</tt> and <tt>pts/dbench</tt> tests.
*   Sysdig Cloud Kubernetes Demo: Starts a kubernetes environment using docker with apache and wordpress instances + synthetic workloads.
*   [Juttle-engine examples][2]: Several elasticsearch, node.js, logstash, mysql, postgres, influxdb instances run under docker-compose.

  
[table id=1 /]This release also has some other handy changes:  
*   Add a new output type "program" that writes a formatted event to a configurable program. Each notification results in one invocation of the program. A common use of this output type would be to send an email for every falco notification.
*   Add the ability to run falco on all events, including events that are flagged with EF_DROP_FALCO. EF_DROP_FALCO events are high-volume, low-value events that are ignored by default to improve performance.For the full set of changes in this release, you can always look at the 

[changelog at github][3].  
  
The release is available via the usual channels–rpm/debian packages, [docker hub][4], [github][5].  
  
Finally, if you want the complete story on Falco, [head over to the website][6] and read all about it.  
  
Let us know if you have any issues, and enjoy!

 [1]: http://www.phoronix-test-suite.com/
 [2]: https://github.com/juttle/juttle-engine/blob/master/examples/README.md
 [3]: https://github.com/draios/falco/blob/master/CHANGELOG.md
 [4]: https://hub.docker.com/r/sysdig/falco
 [5]: https://github.com/draios/falco
 [6]: http://www.sysdig.org/falco/