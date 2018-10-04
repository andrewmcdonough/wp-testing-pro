---
ID: 2672
post_title: Announcing Falco 0.2.0
author: Mark Stemm
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/falco-0-2-0-released/
published: true
post_date: 2016-06-10 16:31:15
---
Today we released version 0.2.0 of Falco. Falco is our new, open source, behavioral security monitoring agent. The major change in this release was a fairly big rework of the ruleset, adding/changing conditions for many rules to improve detection and reduce false positives. We also added a suite of regression tests to ensure stability for future releases.

This ruleset also takes advantage of some new capabilities added to sysdig 0.10.0, specifically session id tracking and the proc.sname filter, to provide a scope for installation-related policies. We’ll be discussing these features in more detail in an upcoming blog post.

For the full set of changes in this release, you can always look at the [changelog at github][1].

The release is available via the usual channels--rpm/debian packages, [docker hub][2], [github][3].

Finally, if you want the complete story on Falco, [head over to the website][4] and read all about it.

Let us know if you have any issues, and enjoy!

 [1]: https://github.com/draios/falco/blob/master/CHANGELOG.md
 [2]: https://hub.docker.com/r/sysdig/falco
 [3]: https://github.com/draios/falco
 [4]: http://www.sysdig.org/falco/