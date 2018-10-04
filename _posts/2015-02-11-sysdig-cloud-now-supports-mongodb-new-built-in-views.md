---
ID: 1135
post_title: 'Sysdig Cloud Now Supports MongoDB, New Built-In Views, &#038; More'
author: Marcus Sarmento
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-cloud-now-supports-mongodb-new-built-in-views/
published: true
post_date: 2015-02-11 06:00:46
---
Our engineering team has been busy building some awesome new functionality and on behalf of the entire company I am thrilled to announce a number of Sysdig Cloud enhancements.  This release is jam-packed with cutting edge functionality that we think will greatly improve visibility into complex environments by introducing brand new metrics and views that automatically correlate application and infrastructure data.  
  
Let’s go ahead and take a closer look at the different improvements we’ve made.  
## Zero-config MongoDB supportWith Sysdig Cloud’s new MongoDB support, our agent now collects and analyzes a number of MongoDB metrics out of the box including:

  
*   net.mongodb.error.count
*   net.mongodb.request.count
*   net.mongodb.request.time
*   net.mongodb.request.time.worstWe’ve also added two new options to group by, net.mongodb.collection and net.mongodb.operation. By providing deep visibility into MongoDB without any additional configuration or installation of plugins, Sysdig Cloud enables ops teams to easily gain access to MongoDB metrics, with the same one-second granularity that users are leveraging to monitor the rest of their environment.

  
  
[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/02/MongoDB-Overview-1024x898.png" alt="MongoDB Overview" width="1024" height="898" class="alignnone wp-image-1142 size-large" />][1]  
  
## Improved process discovery

### Surface command lines with full argumentsIn some cases, process-level information that gets surfaced to operations teams isn’t helpful without additional command line information. For example, if you have Java processes running on a machine, it might not be helpful to see all of those processes rolled up to a single process titled “Java”. Now, in Sysdig Cloud, users can easily view the process command line without having to SSH into the machine to get the next level of detail that can help show what processes are being executed on a particular host in real time or in the past.  Think of this as having top for any of your machines, or groups of machines, from an explorable centralized interface.

  
  
[][2][][3][<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/02/process-command-line2-1024x347.png" alt="process command line2" width="800" height="271" class="alignnone wp-image-1171" />][4]  
### Ability to group data by process name or by command lineBuilding on the process command line capability, users now have the ability to group any view or table that contains process data in it by process name or by process command line. By providing full control of how users view and consume process-level information, Sysdig Cloud allows users to quickly iterate between different views to uncover the information they need.

  
  
[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/02/process-name-command-line-grouping2-1024x195.png" alt="process name & command line grouping2" width="500" height="95" class="alignnone wp-image-1170" />][5][  
][6]  
## New built-in views

### Network traffic overviewThe network traffic overview exposes network metrics for components of the infrastructure. With this view, Sysdig Cloud users can now get visibility into in vs out network bytes, network bytes by application, network bytes by process, and network bytes by host.  Monitoring network traffic activity is a key part of managing modern infrastructure environments, and instead of having to pull metrics from silo'd tools and manually correlate them, Sysdig Cloud allows users to explore this information alongside system and application metrics all in one unified view.  This improved visibility empowers operations to see network metrics in the context of a particular host machine, tier, availability zones, regions, etc. to gain yet another way of analyzing system performance.

  
  
[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/02/network-traffic-overview2-1024x462.png" alt="network traffic overview2" width="1024" height="462" class="alignnone wp-image-1169 size-large" />][7]  
### Response time mapWith the response time map, users can now visualize and measure application response times without needing to instrument the application code. This topological view of the requests flowing through the environment enables ops teams to understand the end user experience without needing to configure or install any additional agents or having to manually correlate this data coming from another tool.

  
  
[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/02/response-time-map2-1024x210.png" alt="response time map2" width="1024" height="210" class="alignnone wp-image-1168 size-large" />][8]  
### Response time vs resource usageThe response time vs resource usage view shows response time correlated with CPU utilization %, memory utilization %, average network bytes, and average file bytes. Automatically correlating system, network, and application metrics lets users see the underlying cause of spikes in response times. Being able to quickly pinpoint hardware issues behind potential bottlenecks should drastically reduce the time spent searching for the root cause of performance issues.

  
  
[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/02/response-time-vs-resource-usage2-1024x212.png" alt="response time vs resource usage2" width="1024" height="212" class="alignnone wp-image-1167 size-large" />][9]  
  
## New setup wizard and video walkthroughsWhether you are new to Sysdig Cloud or are an existing user, we’re confident you’ll gain value out of our new setup wizard and detailed video walkthroughs. We are always focused on delivering value to customers, and this feature surfaces the information you need to start monitoring your environment or install additional agents across your infrastructure in minutes.

  
  
[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/02/setup-wizard2-1024x919.png" alt="setup wizard2" width="500" height="449" class="alignnone wp-image-1173" />][10]  
[][11]  
If you haven’t already done so, I encourage you to sign up for a [trial of Sysdig Cloud][12] to try these new features out for yourself!

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/02/MongoDB-Overview.png
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/02/process-command-line1.png
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/02/process-command-line.png
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/02/process-command-line2.png
 [5]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/02/process-name-command-line-grouping2.png
 [6]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/02/process-name-command-line.png
 [7]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/02/network-traffic-overview2.png
 [8]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/02/response-time-map2.png
 [9]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/02/response-time-vs-resource-usage2.png
 [10]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/02/setup-wizard2.png
 [11]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/02/startup-wizard.png
 [12]: https://sysdigrp2rs.wpengine.com/sign-up/?utm_campaign=blog_021115&utm_medium=blogCTA&utm_source=blog&utm_content=&utm_term=