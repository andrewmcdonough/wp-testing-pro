---
ID: 2454
post_title: >
  Correlating alerts in container
  environments
author: Apurva Dave
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/correlating-alerts-container-environments/
published: true
post_date: 2016-03-16 09:19:58
---
Data correlation is perhaps one of the hardest parts of troubleshooting in a complex distributed software environment. It’s relatively easy to set-up your APM tool, your monitoring tool, your logging tool, your synthetic monitoring tool and so on to capture data and even alert on problems. It of course gets harder in container environments where these traditional tools don’t work, but we’ll ignore that point for the moment.

When it comes down to actually solving the problem, all those tools are part of the problem: you’ll find yourself cut-and-pasting a dozen graphs into a doc and trying to give your team a view into the right data. But you ideally want to have all that information in the same application, with a sane way to correlate all of those data types.

Today we’re making a small step towards making data correlation in container environments much, much simpler with notification overlays. This is the first step in a longer term strategy for Sysdig to not only provide you powerful metrics analysis, but also provide you the ability to work with semi-structured data within your overall operations analysis and monitoring practice. 

Notification overlays allow you to correlate semi-structured alert data with your metrics, and selectively look at alerts within a broad or narrow context. As always, a picture is worth a thousand data points:

![Notifications Overlay][1]

As you see, we allow you to correlate Sysdig-fired alerts with your metrics. The beauty is we let you do it with the flip of a switch:

![Notifications Overlay][2]

By simply choosing the notification icon on any dashboard or and panel in the explore view, you can show notifications. You make two main choices: 
*   Environment scope: Choose notifications only associated with the metric or nodeyou’re looking at, or choose a broader scope in either direction
*   Notification characteristics: Choose notifications based on their state (Active/OK), severity, or resolution.

These choices allow you to do some pretty interesting things. For example, in the dashboard below, I can correlate the response time of my Web service with high throughput alerts from my Mongo service:

![Notifications Overlay][3]

### The future for correlations and Sysdig 

Today we’re letting you visually correlate Sysdig alerts with Sysdig metrics. The feature is valuable, and it speaks to something much larger.

An alert is simply a type of semi-structured data structure. We’re already experimenting with ways to bring in different types of events, and we want to make it easy for you to correlate those other events with the hundreds and hundreds of metrics that Sysdig pulls in from your containers, apps, and clouds. Stay tuned for more!

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2016/03/Notification-Overlay-3.png
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2016/03/Notification-Overlay-2.png
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2016/03/Notification-Overlay-1.png