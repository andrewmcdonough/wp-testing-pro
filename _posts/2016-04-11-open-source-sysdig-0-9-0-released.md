---
ID: 2528
post_title: Open source Sysdig 0.9.0 released!
author: Apurva Dave
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/open-source-sysdig-0-9-0-released/
published: true
post_date: 2016-04-11 11:34:51
---
We’re excited to announce another release of our open source workhorses, sysdig and csysdig.

The big news is native support for monitoring Mesos, Marathon, and DCOS. You can read in detail about how it works in our [blog post here][1]. In a nutshell we now provide the following:

*   `-m sysdig/csysdig` parameter to specify URLs for Mesos Master and Marathon API. We collect metadata in order to create Mesos/Marathon specific views.
*   csysdig views: `Mesos Tasks, Mesos Frameworks, Marathon Apps, Marathon Groups`
*   -pm sysdig parameter to get a Mesos-friendly event output
*   Filter fields: `mesos.task.name, mesos.task.id, mesos.task.label, mesos.task.labels,mesos.framework.name, mesos.framework.id, marathon.app.name, marathon.app.id,marathon.app.label, marathon.app.labels, marathon.group.name, marathon.group.id`

Next, on the network monitoring side:

You asked, we delivered! You can now create subnet-based filters in sysdig using CIDR notation, giving you a more powerful and natural way to get to the specific data you need.

*   New network filter fields that support a CIDR notation (e.g. 127.0.0.1/24): `fd.net, fd.cnet,fd.snet, fd.lnet, fd.rnet`

We’ve also improved our Kubernetes authentication support:

*   Support for SSL based authentication and bearer token authentication against the Kubernetes API server. Previously, SSL was just supported for CA verification. See the updated documentation for `-K`

And finally just some general improvements

*   New actions on csysdig views: `lsof` and `renice`
*   `icontains` filter comparison operator: case-insensitive string comparison

Download sysdig from [www.sysdig.org][2] or grab the source from [Github][3].

Happy digging!

 [1]: https://sysdigrp2rs.wpengine.com/blog/monitoring-mesos
 [2]: http://sysdig.org
 [3]: https://github.com/draios/sysdig