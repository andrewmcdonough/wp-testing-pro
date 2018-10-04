---
ID: 2158
post_title: Announcing Sysdig 0.3.0
author: Gianluca Borello
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/announcing-sysdig-0-3-0/
published: true
post_date: 2015-10-13 18:35:15
---
**New and updated features** 

*   Support ia32 emulation on 64bit kernels, now you can finally dig into skype!
*   New events: `mount`, `umount`
*   New `memcachelog` chisel: show a log of memcached commands (get/set)
*   New `backlog` csysdig view: show queues (backlog) utilization per process
*   `--unbuffered` command line option: turn off output buffering
*   Detect `mesos` containers (support is still limited)
*   HTTP chisels now support UNIX sockets (e.g. Docker API)
*   New section in the csysdig views files: hotkeys

**Bug Fixes** 

*   Minor bugfixes

  
## Downloads

*   <a href="https://github.com/draios/sysdig/archive/0.2.0.zip" target= "_blank">**Source code** (zip)</a> 
*   <a href="https://github.com/draios/sysdig/archive/0.2.0.tar.gz" target= "_blank">**Source code** (tar.gz)</a> 

## Sources

[Release details][1] 

<a href= "https://github.com/draios/sysdig/wiki/Sysdig%20Update%20and%20Uninstall">Update instructions</a> 

[Installation instructions][2] 

[Source code][3] 

## Support

Community support is available on the <a href= "https://groups.google.com/forum/#!forum/sysdig">sysdig mailing list</a>.

Bugs and issues can be <a href= "https://github.com/draios/sysdig/issues?state=open">submitted through github</a>.

 [1]: https://github.com/draios/sysdig/releases
 [2]: http://www.sysdig.org/install/
 [3]: https://github.com/draios/sysdig