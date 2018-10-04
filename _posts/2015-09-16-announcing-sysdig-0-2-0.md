---
ID: 2055
post_title: Announcing Sysdig 0.2.0
author: Gianluca Borello
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/announcing-sysdig-0-2-0/
published: true
post_date: 2015-09-16 07:00:08
---
Even if we're still on the 0.x series of sysdig, starting from this release we are adopting semantic versioning (<a href="http://semver.org/" target="_blank">http://semver.org/</a>) so it will be easier to identify bug fix releases. 

## Bug Fixes 

*   Support Debian 7 as a host for the sysdig Docker container
*   Minor bugfixes

## New and updated features 

*   Port numbers will be automatically converted to service names (according to the services file on your platform) unless `-N` is specified
*   New filter field `fd.proto`: matches the protocol (either client or server) of the fd
*   New filter field `fd.cproto`: for TCP/UDP FDs, the client protocol
*   New filter field `fd.sproto`: for TCP/UDP FDs, server protocol
*   New filter field `fd.lproto`: for TCP/UDP FDs, the local protocol
*   New filter field `fd.rproto`: for TCP/UDP FDs, the remote protocol
*   New events: `semop`, `semctl`, `ppoll`
*   Docker image now includes the `RUN` label to make it easier to run sysdig on Atomic Linux

## Downloads 

*   <a href="https://github.com/draios/sysdig/archive/0.2.0.zip" target="_blank"><b>Source code</b> (zip)</a>
*   <a href="https://github.com/draios/sysdig/archive/0.2.0.tar.gz" target="_blank"><b>Source code</b> (tar.gz)</a>

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