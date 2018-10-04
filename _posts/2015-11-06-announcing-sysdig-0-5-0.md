---
ID: 2220
post_title: Announcing Sysdig 0.5.0
author: Gianluca Borello
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/announcing-sysdig-0-5-0/
published: true
post_date: 2015-11-06 16:08:27
---
## New and updated features

*   Full Kubernetes support!
*   `--k8s-api` command line option: specify the Kubernetes API server endpoint
*   `-pk`: Kubernetes-friendly output format

## New display/filter fields

*   `k8s.pod.name`: Kubernetes pod name.
*   `k8s.pod.id`: Kubernetes pod id.
*   `k8s.pod.label`: Kubernetes pod label. E.g. 'k8s.pod.label.foo'.
*   `k8s.pod.labels`: Kubernetes pod comma-separated key/value labels. E.g. 'foo1:bar1,foo2:bar2'.
*   `k8s.rc.name`: Kubernetes replication controller name.
*   `k8s.rc.id`: Kubernetes replication controller id.
*   `k8s.rc.label`: Kubernetes replication controller label. E.g. 'k8s.rc.label.foo'.
*   `k8s.rc.labels`: Kubernetes replication controller comma-separated key/value labels. E.g. 'foo1:bar1,foo2:bar2'.
*   `k8s.svc.name`: Kubernetes service name (can return more than one value, concatenated).
*   `k8s.svc.id`: Kubernetes service id (can return more than one value, concatenated).
*   `k8s.svc.label`: Kubernetes service label. E.g. 'k8s.svc.label.foo' (can return more than one value, concatenated).
*   `k8s.svc.labels`: Kubernetes service comma-separated key/value labels. E.g. 'foo1:bar1,foo2:bar2'.
*   `k8s.ns.name`: Kubernetes namespace name.
*   `k8s.ns.id`: Kubernetes namespace id.
*   `k8s.ns.label`: Kubernetes namespace label. E.g. 'k8s.ns.label.foo'.
*   `k8s.ns.labels`: Kubernetes namespace comma-separated key/value labels. E.g. 'foo1:bar1,foo2:bar2'.

## New csysdig views

*   Kubernetes Controllers
*   Kubernetes Namespaces
*   Kubernetes Pods
*   Kubernetes Services

## Misc

*   Add a convenient `USE_BUNDLED_DEPS` CMake option to enable/disable all bundled dependencies at once.
*   New build/runtime dependencies: `libb64`, `libcurl`, `openssl`.

## Known issues

*   The Kubernetes state is not yet serialized to a trace file, this will come over the next release. Thus, if you take a trace file, be sure to still use `-k` in conjunction with `-r` to make sure the Kubernetes data is fetched from the API server when reading it.

## Downloads

*   <a href="https://github.com/draios/sysdig/archive/0.5.0.zip" target="_blank";>**Source code** (zip)</a>
*   <a href="https://github.com/draios/sysdig/archive/0.5.0.tar.gz" target="_blank";>**Source code** (tar.gz)</a>

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