---
ID: 1035
post_title: Announcing Sysdig 0.1.94
author: Gianluca Borello
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/announcing-sysdig-0-1-94/
published: true
post_date: 2015-01-20 16:34:46
---
**Bug Fixes  
** 
*   Improved performance of `libscap` during live captures
*   Expand the `sysdig-probe` ring buffer size: now it's 8 MB per CPU (was 1 MB)
*   Fix syntax error in `fdtime_by chisel`
*   Fix syntax error in `zsh` completion
*   Many minor bugfixes
*   `evt.latency` was returning wrong values

**New and updated features**

<ul class="task-list">
  <li>
    <code>get_terminal_info</code> chisel API function
  </li>
</ul>

**New and updated chisels**

<ul class="task-list">
  <li>
    <code>spectrogram</code> chisel: Visualize OS latency in real time.
  </li>
</ul>

**New and updated filter fields**

<ul class="task-list">
  <li>
    <code>evt.buflen</code> filter field
  </li>
</ul>

**New and Updated events**

<ul class="task-list">
  <li>
    <code>clone</code>, <code>execve</code>, <code>fork</code>, <code>vfork</code>: return <code>comm</code> parameter to report the executable file name rather than guessing it from <code>argv[0]</code>
  </li>
</ul>

## Resources

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