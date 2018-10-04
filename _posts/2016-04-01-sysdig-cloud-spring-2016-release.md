---
ID: 2499
post_title: Sysdig Monitor spring 2016 release
author: Apurva Dave
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-cloud-spring-2016-release/
published: true
post_date: 2016-04-01 10:36:13
---
We’re really excited to round up all the great functionality we’ve released so far this year on Sysdig Monitor into our Spring Release. Here we’re going to give you a quick run-down on the most important features and where possible link you to deeper information on how to use the features.

If you’ve got questions on any of these features, or want to assistance using them, we’d love to help you! Drop us a line at <a href="https://twitter.com/sysdig" target="_blank">@sysdig</a>.

### Increased performance and scale

Sysdig Monitor now supports tens of thousands of containers per customer, with 10x faster dashboard load time. More data, faster!

### Added Container Ecosystem Support

<a href="/wp-content/uploads/2016/02/rkt-logo.png" rel="attachment wp-att-2397"><img src="/wp-content/uploads/2016/02/rkt-logo.png" alt="rkt logo" width="100" height="100" align="left" style="padding-right: 15px;" class="" /></a>
**[CoreOS rkt support][1]** - Sysdig now support rkt containers just like we do Docker containers. Same great monitoring and alerting, now with the ultra-secure rkt container from CoreOS. Go deeper with our blog post, or read the CoreOS rkt release post.

<a href="/wp-content/uploads/2015/07/Docker.png" rel="attachment wp-att-2397"><img src="/wp-content/uploads/2015/07/Docker.png" alt="docker logo" width="100" height="100" align="left" style="padding-right: 15px;" class="" /></a>
**[Sysdig Integrates with Docker Data Center][2]** - Sysdig Monitor can capture key metadata (based on Docker Labels) from Docker DataCenter and use it to monitor the logical services of your environment.

<a href="/wp-content/uploads/2015/07/Kubernetes.png" rel="attachment wp-att-2397"><img src="/wp-content/uploads/2015/07/Kubernetes.png" alt="Kubernetes logo" width="100" height="100" align="left" style="padding-right: 15px;" class="" /></a>
**[Sysdig supports Kubernetes 1.2 out-of-the-box][3]** - Kubernetes Scale, Ubernetes Lite, and DeamonSets are all supported within Sysdig.

**New Views** - We added new views for AWS ECS, Docker, and Kubernetes that allow you to better monitor your environment out-of-the-box.

### Deeper Troubleshooting

**[Sysdig Monitor now lets you troubleshoot a container after it’s long gone][4]**. Alerts can now trigger a system capture. Most monitoring tools stop at a dashboard and an alert. But we let you go deeper, using sysdig’s open source tool. Any alert can now trigger a system capture, recording every syscall made on a container or host at the time the alert was fired into a file.

Sysdig graphs now **<a href="https://sysdigrp2rs.wpengine.com/blog/correlating-alerts-container-environments/" target="_blank">correlate alerts</a> **with metrics - Get simple, beautiful alert overlays on any graph. It makes getting to the heart of any problem much easier and much more pleasant.

<a href="/wp-content/uploads/2016/03/Notification-Overlay-3.png" rel="attachment wp-att-2397"><img src="/wp-content/uploads/2016/03/Notification-Overlay-3.png" alt="Notification Overlay" width="750" height="536" align="left" style="padding-right: 15px;" /></a>

 

### Better Monitoring and Analytics

**<a href="https://sysdigrp2rs.wpengine.com/blog/better-container-monitoring/" target="_blank">Number panel</a>** - This was a highly requested feature. A number panel shows you the state of a metric “right now”, and dynamic color coding ensures that your wallboard will always be really useful and you’re running by it.

<a href="/wp-content/uploads/2016/02/Number-Panel-Blog-1.png" rel="attachment wp-att-2397"><img src="/wp-content/uploads/2016/02/Number-Panel-Blog-1.png" alt="Number Panels" width="750" height="536" align="left" style="padding-right: 15px;" /></a>

 

**<a href="https://sysdigrp2rs.wpengine.com/blog/no-plugins-required-application-visibility-inside-containers/" target="_blank">New Integrations</a>** - We added 20+ application views, which give you templated instant data on components that you might use in your app or service today. Combined with Sysdig’s auto-app discovery capability, it means that you can install sysdig and immediately monitor your app with no config necessary.

<a href="/wp-content/uploads/2016/03/Logo-Soup.png" rel="attachment wp-att-2397"><img src="/wp-content/uploads/2016/03/Logo-Soup.png" alt="Integrations" width="750" height="536" p="" /><b></b></a>

 

**Customizable columns in the Explore view **- Customers love our explore view because it’s like a big pivot table for their entire infrastructure. Grouping features have always provided a way to switch between logical and physcial views of your apps, but data columns had been statically configured, until now. The explore table now allows you to configure every column for the exact metric you’d like to see.

**Dashboard Wizard** - well, this one seems pretty self explanatory. But still awesome. So awesome, in fact, that since we introduced it we’ve seen a 10x increase in the number of dashboards created!

 [1]: https://sysdigrp2rs.wpengine.com/blog/monitoring-rkt-sysdig/
 [2]: https://sysdigrp2rs.wpengine.com/blog/docker-monitoring-with-datacenter/
 [3]: https://sysdigrp2rs.wpengine.com/blog/3-ways-that-kubernetes-changes-monitoring/
 [4]: https://sysdigrp2rs.wpengine.com/blog/troubleshooting-containers-when-theyre-gone/