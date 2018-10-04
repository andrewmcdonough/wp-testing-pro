---
ID: 1440
post_title: Announcing Sysdig 0.1.98
author: Gianluca Borello
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/announcing-sysdig-0-1-98/
published: true
post_date: 2015-03-02 12:16:25
---
**Bug Fixes**

<ul class="task-list">
  <li>
    Many minor bugfixes
  </li>
</ul>

**New and updated features**

<ul class="task-list">
  <li>
    Container support: sysdig now supports Docker, LXC and libvirt-lxc containers, with several sub-features described below and in the documentation
  </li>
  <li>
    supports to an alternate <code>/proc</code> file system tree (useful in containers) by setting the environment variable <code>SYSDIG_HOST_ROOT</code>
  </li>
  <li>
    supports parsing network connections from <code>/proc</code> from a network namespace different than the global one
  </li>
  <li>
    container information is available in the chisel API (thread table)
  </li>
  <li>
    <code>-pc</code> and <code>-pcontainer</code> will use a container-friendly output format for events
  </li>
  <li>
    Automated Docker builds for running sysdig:<a href="https://registry.hub.docker.com/u/sysdig/sysdig/">https://registry.hub.docker.com/u/sysdig/sysdig/</a>
  </li>
  <li>
    <code>sysdig-probe-loader</code>: new script included with sysdig to facilitate loading the <code>sysdig-probe</code>module in atypic environments such as containers
  </li>
  <li>
    <code>build-sysdig-probe-binaries</code>: new script to prebuild <code>sysdig-probe</code> binaries for a specific set of kernel configurations (currently CoreOS) and upload them to S3 so that they can be downloaded at runtime on environments that don't ship kernel headers
  </li>
</ul>

**New and updated chisels**

<ul class="task-list">
  <li>
    <code>lscontainers</code>: List the running containers.
  </li>
  <li>
    <code>topcontainers_cpu</code>: Top containers by CPU usage.
  </li>
  <li>
    <code>topcontainers_error</code>: Top containers by number of errors.
  </li>
  <li>
    <code>topcontainers_file</code>: Top containers by R+W disk bytes.
  </li>
  <li>
    <code>topcontainers_net</code>: Top containers by network I/O.
  </li>
  <li>
    <code>echo_fds</code>: container-aware (with <code>-pc</code>).
  </li>
  <li>
    <code>fileslower</code>: container-aware (with <code>-pc</code>).
  </li>
  <li>
    <code>list_login_shells</code>: container-aware (with <code>-pc</code>).
  </li>
  <li>
    <code>netlower</code>: container-aware (with <code>-pc</code>).
  </li>
  <li>
    <code>proc_exec_time</code>: container-aware (with <code>-pc</code>).
  </li>
  <li>
    <code>scallslower</code>: container-aware (with <code>-pc</code>).
  </li>
  <li>
    <code>spy_logs</code>: container-aware (with <code>-pc</code>).
  </li>
  <li>
    <code>spy_syslog</code>: container-aware (with <code>-pc</code>).
  </li>
  <li>
    <code>spy_users</code>: container-aware (with <code>-pc</code>).
  </li>
  <li>
    <code>stderr</code>: container-aware (with <code>-pc</code>).
  </li>
  <li>
    <code>topconns</code>: container-aware (with <code>-pc</code>).
  </li>
  <li>
    <code>topfiles_bytes</code>: container-aware (with <code>-pc</code>).
  </li>
  <li>
    <code>topfiles_errors</code>: container-aware (with <code>-pc</code>).
  </li>
  <li>
    <code>topfiles_time</code>: container-aware (with <code>-pc</code>).
  </li>
  <li>
    <code>topports_server</code>: container-aware (with <code>-pc</code>).
  </li>
  <li>
    <code>topprocs_cpu</code>: container-aware (with <code>-pc</code>).
  </li>
  <li>
    <code>topprocs_errors</code>: container-aware (with <code>-pc</code>).
  </li>
  <li>
    <code>topprocs_file</code>: container-aware (with <code>-pc</code>).
  </li>
  <li>
    <code>topprocs_net</code>: container-aware (with <code>-pc</code>).
  </li>
  <li>
    <code>topscalls</code>: container-aware (with <code>-pc</code>).
  </li>
  <li>
    <code>topscalls_time</code>: container-aware (with <code>-pc</code>).
  </li>
</ul>

**New and updated filter fields**

<ul class="task-list">
  <li>
    <code>thread.cgroups</code>: all the cgroups the thread belongs to, aggregated into a single string.
  </li>
  <li>
    <code>thread.cgroup</code>: the cgroup the thread belongs to, for a specific subsystem. E.g.<code>thread.cgroup.cpuacct</code>.
  </li>
  <li>
    <code>thread.vtid</code>: the id of the thread generating the event as seen from its current PID namespace.
  </li>
  <li>
    <code>proc.vpid</code>: the id of the process generating the event as seen from its current PID namespace.
  </li>
  <li>
    <code>container.id</code>: the container id.
  </li>
  <li>
    <code>container.name</code>: the container name.
  </li>
  <li>
    <code>container.image</code>: the container image.
  </li>
</ul>

**New and Updated events**

<ul class="task-list">
  <li>
    <code>clone</code>, <code>execve</code>, <code>fork</code>, <code>vfork</code>: add <code>cgroups</code>, <code>vtid</code> and <code>vpid</code> to the events to correctly report control group and PID namespaces information.
  </li>
</ul>

  
<span>A blog post with an in-depth look at this new functionality will be published very soon.  Stay tuned!</span>  
<h2 class="release-downloads-header">
  Downloads
</h2>

<ul class="release-downloads">
  <li>
    <a href="https://github.com/draios/sysdig/archive/0.1.98.zip" rel="nofollow"><span class="octicon octicon-file-zip text-muted"></span> <strong>Source code</strong> (zip)</a>
  </li>
  <li>
    <a href="https://github.com/draios/sysdig/archive/0.1.98.tar.gz" rel="nofollow"><span class="octicon octicon-file-zip text-muted"></span> <strong>Source code</strong> (tar.gz)</a>
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