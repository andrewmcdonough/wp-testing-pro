---
ID: 306
post_title: Announcing Sysdig 0.1.87
author: Gianluca Borello
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/announcing-sysdig-0-1-87/
published: true
post_date: 2014-08-06 15:20:24
---
The highlight of this release is a bunch of cool new functionality for using sysdig with logs, including two new/updated chisels: 
*   `spy_logs`: This chisel intercepts all the writes to files containing `.log` or `_log` in their name, and pretty prints them
*   `spy_syslog` (was `echo_syslog`): Print every message written to syslog We have a longer blog post coming out tomorrow that will show off this new functionality in detail, so stay tuned! 

## Resources

[Release details][1]

[Update instructions][2]

[Installation instructions][3]

[Source code][4]

## Support

Community support is available on the [sysdig mailing list][5].

Bugs and issues can be [submitted through github][6].

 [1]: https://github.com/draios/sysdig/releases
 [2]: https://github.com/draios/sysdig/wiki/Sysdig%20Update%20and%20Uninstall
 [3]: http://www.sysdig.org/install/
 [4]: https://github.com/draios/sysdig
 [5]: https://groups.google.com/forum/#!forum/sysdig
 [6]: https://github.com/draios/sysdig/issues?state=open