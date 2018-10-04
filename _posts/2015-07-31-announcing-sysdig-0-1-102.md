---
ID: 1976
post_title: Announcing Sysdig 0.1.102
author: Gianluca Borello
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/announcing-sysdig-0-1-102/
published: true
post_date: 2015-07-31 06:50:47
---
## Bug Fixes 

*   Fix panic with some kernel versions
*   Fix compiling errors on arm architecture
*   Report `execve` args even if it fails
*   Minor bugfixes on csysdig

## New and updated features 

*   Support for decoding `setns` and `flock` syscall
*   Parse `O_CLOEXEC` flag on `open` and related syscalls
*   Parse `CLONE_NEWUSER` flag on `clone`
*   Support truncated tracefiles
*   Now sysdig can rotate tracing file when capturing, using `-C`, `-e`, `-W`, `-G`
*   Better extraction/filtering capabilities for event related to multiple file descriptors, like `poll`
*   Precompiled kernel modules for older coreos releases

## Downloads 

*   <a href="https://github.com/draios/sysdig/archive/0.1.102.zip" target="_blank"><b>Source code</b> (zip)</a>
*   <a href="https://github.com/draios/sysdig/archive/0.1.102.tar.gz" target="_blank"><b>Source code</b> (tar.gz)</a>

## Sources

[Release details][1]

[Update instructions][2]

[Installation instructions][3]

[Source code][4]

## Support

Community support is available on the [sysdig mailing list][5].

Bugs and issues can be [submitted through github][6].

 [1]: https://github.com/draios/sysdig/releases
 [2]: https://github.com/draios/sysdig/wiki/Sysdig%20Update%20and%20Uninstall
 [3]: http://www.sysdig.org/install/
 [4]: https://github.com/draios/sysdig
 [5]: https://groups.google.com/forum/#!forum/sysdig
 [6]: https://github.com/draios/sysdig/issues?state=open