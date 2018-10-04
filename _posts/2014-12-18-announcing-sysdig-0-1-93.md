---
ID: 1009
post_title: Announcing Sysdig 0.1.93
author: Gianluca Borello
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/announcing-sysdig-0-1-93/
published: true
post_date: 2014-12-18 09:25:28
---
<span class="s1"><b>Bug Fixes<br /></b></span> 
*   Fixed memory leaks
*   <span class="s1">Many minor bugfixes</span>

<p class="p1">
  <span class="s1"><b>New and Updated Features</b></span>
</p>

<ul class="ul1">
  <li class="li1">
    <span class="s2"></span><span class="s3"><tt>-E/--exclude-users</tt></span><span class="s1"> command line flag, to prevent importing user and group tables when the capture starts</span>
  </li>
  <li class="li1">
    <span class="s1">Buffer rendering on screen is now limited to 80 bytes unless </span><span class="s3">-v</span><span class="s1"> is used</span>
  </li>
</ul>

<p class="p1">
  <span class="s1"><b>New and Updated Chisels</b></span>
</p>

<ul class="ul1">
  <li class="li1">
    <span class="s1">Process name is printed in </span><span class="s3"><tt>echo_fds</tt></span><span class="s1"> header</span>
  </li>
</ul>

<p class="p1">
  <span class="s1"><b>New and Updated events</b></span>
</p>

<ul class="ul1">
  <li class="li1">
    <span class="s2"></span><span class="s3"><tt>procexit</tt></span><span class="s1"> now returns the thread exit code</span>
  </li>
  <li class="li2">
    <span class="s1"><tt>sendfile</tt></span>
  </li>
  <li class="li2">
    <span class="s1"><tt>quotactl</tt></span>
  </li>
  <li class="li2">
    <span class="s1"><tt>setresuid</tt></span>
  </li>
  <li class="li2">
    <span class="s1"><tt>setresgid</tt></span>
  </li>
  <li class="li2">
    <span class="s1"><tt>setuid</tt></span>
  </li>
  <li class="li2">
    <span class="s1"><tt>setgid</tt></span>
  </li>
  <li class="li2">
    <span class="s1"><tt>getgid</tt></span>
  </li>
  <li class="li2">
    <span class="s1"><tt>getegid</tt></span>
  </li>
  <li class="li2">
    <span class="s1"><tt>getuid</tt></span>
  </li>
  <li class="li2">
    <span class="s1"><tt>geteuid</tt></span>
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