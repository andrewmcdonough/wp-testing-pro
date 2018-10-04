---
ID: 2003
post_title: Announcing Sysdig 0.1.103
author: Gianluca Borello
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/announcing-sysdig-0-1-103/
published: true
post_date: 2015-08-24 09:00:46
---
This release includes several new interesting features. Sysdig now supports RHEL Atomic Host, CentOS Atomic Host and Fedora Atomic Host. We also added a couple of nice chisels that record and display HTTP requests and are going to be useful when monitoring web servers. Finally, the sysdig driver is now CPU hotplug friendly, which can be useful in production environments and workloads. 

## Bug Fixes 

*   Update `ncurses` so it will compile on GCC 5
*   Don't use GCC 5 inside the Docker container, because older kernels are still not ready
*   Minor bugfixes on csysdig

## New and updated features 

*   `httplog` chisel: show a log of all HTTP requests
*   `httptop` chisel: show top HTTP requests by: ncalls, time or bytes
*   Improved the `accept` system event by adding `queuelen` and `queuemax`
*   `sysdig-probe` can now compile on the EL5 kernel. Userspace application still requires a recent GCC, which can be obtained from the Redhat/CentOS developer toolset
*   Support CPU hotplug: sysdig will just work if CPUs go up or down in your system, and will also generate an event when that happens
*   Precompile `sysdig-probe` for most Ubuntu, Fedora, CentOS kernels

## Downloads 

*   <a href="https://github.com/draios/sysdig/archive/0.1.103.zip" target="_blank"><b>Source code</b> (zip)</a>
*   <a href="https://github.com/draios/sysdig/archive/0.1.103.tar.gz" target="_blank"><b>Source code</b> (tar.gz)</a>

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