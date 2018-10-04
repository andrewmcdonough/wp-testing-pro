---
ID: 1532
post_title: Announcing Sysdig 0.1.99
author: Gianluca Borello
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/announcing-sysdig-0-1-99/
published: true
post_date: 2015-03-16 17:01:22
---
<h2 class="release-downloads-header">
  Bug Fixes
</h2>

<ul class="task-list">
  <li>
    Under certain conditions, sysdig could crash during socket scanning in <code>/proc</code>
  </li>
  <li>
    Improve default truncation algorithm when <code>-v</code> is not specified
  </li>
  <li>
    Improved <code>spy_users</code> chisel accuracy
  </li>
  <li>
    Many minor bugfixes
  </li>
</ul>

<h2 class="release-downloads-header">
  New and updated features
</h2>

<ul class="task-list">
  <li>
    sysdig can now be concurrently opened multiple times
  </li>
  <li>
    <code>exists</code> clause for filters, e.g. <code>sysdig proc.name exists</code>
  </li>
  <li>
    <code>in</code> clause for filters, e.g. <code>sysdig "evt.type in ( 'select', 'poll' )"</code>
  </li>
</ul>

<h2 class="release-downloads-header">
  Downloads
</h2>

<ul class="release-downloads">
  <li>
    <a href="https://github.com/draios/sysdig/archive/0.1.99.zip" rel="nofollow"><span class="octicon octicon-file-zip text-muted"></span> <strong>Source code</strong> (zip)</a>
  </li>
  <li>
    <a href="https://github.com/draios/sysdig/archive/0.1.99.tar.gz" rel="nofollow"><span class="octicon octicon-file-zip text-muted"></span> <strong>Source code</strong> (tar.gz)</a>
  </li>
</ul>

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