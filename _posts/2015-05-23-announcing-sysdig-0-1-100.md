---
ID: 1591
post_title: Announcing Sysdig 0.1.100
author: Gianluca Borello
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/announcing-sysdig-0-1-100/
published: true
post_date: 2015-05-23 14:14:11
---
## Bug Fixes 

*   Many minor bugfixes
*   Docker container ships with GCC 4.8 other than the latest from Debian, to increase compatibility
*   `echo_fds` chisel has a better formatting
*   Correctly show container output even when renaming containers on Docker >= 1.5
*   Fixes on the `exists` filter operator

## New and updated features 

*   Support for intercepting signals via the `signaldeliver` event: parameters are source pid, destination pid and signal type

## Downloads 

*   <a href="https://github.com/draios/sysdig/archive/0.1.100.zip" target="_blank"><b>Source code</b> (zip)</a>
*   <a href="https://github.com/draios/sysdig/archive/0.1.100.tar.gz" target="_blank"><b>Source code</b> (tar.gz)</a>

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