---
ID: 2190
post_title: 'Sysdig Cloud and PagerDuty: a Superior Alerting Experience'
author: Chris Crane
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-cloud-and-pagerduty-a-superior-alerting-experience/
published: true
post_date: 2015-11-05 09:00:27
---
One of the most common ways that users interact with Sysdig Cloud (and all monitoring tools, really) is through alerts. You get a notification that something is wrong - hopefully before you hear about it from your users - and you need to go investigate. We’ve worked hard to make this alert-response experience as smooth and intuitive as possible in Sysdig Cloud, and we take pride that our customers report massive improvements in mean time to resolution when they’re troubleshooting (think 10X+). 

Along those lines, our integration with PagerDuty is one of the most used integrations within Sysdig Cloud. By hooking up your Sysdig Cloud account to your PagerDuty account, you can automatically route Sysdig Cloud notifications directly through PagerDuty. This lets you manage your on-call duty and route all your alerts accordingly, from one central location. 

<a href="/wp-content/uploads/2015/10/New-Alert-Configure-with-PagerDuty.png" data-rel="lightbox-0" title> <img src="/wp-content/uploads/2015/10/New-Alert-Configure-with-PagerDuty.png" class="aligncenter size-full" /> </a> 

Today I’m pleased to announce that we’ve taken our integration with PagerDuty one step further by linking notification status across the two apps. Now, Sysdig Cloud doesn’t just push notifications to PagerDuty, we’ll also automatically keep the status of that notification up to date! 

Status of notifications in Sysdig Cloud is tracked in two ways: 

*   First off, if a threshold condition resolves to an acceptable state, then a notification will automatically be moved to an “OK” status. This way you still have a record of the issue, but you know that your system is no longer in an alert-worthy state. 
*   Second, you can manually resolve any notification as well. This way, if you’ve investigated a situation, or are otherwise not concerned for any reason, you can dismiss the notification, and clear up your alert dashboard. 

You can of course sort all of your notifications by status within Sysdig Cloud, so that only the most important will filter to the top: 

<a href="/wp-content/uploads/2015/10/Alerting-Notifications.png" data-rel="lightbox-0" title> <img src="/wp-content/uploads/2015/10/Alerting-Notifications.png" class="aligncenter size-full" /> </a> 

And now, with this new feature release, if any notification is resolved in Sysdig Cloud, we will automatically push the Resolved status of the alert to PagerDuty. This way you can keep your PagerDuty notifications list clean, and not have to interface with the same notification in multiple places. 

The design of this feature is based on conversations we’ve had with many current Sysdig Cloud users about their preferred workflows with PagerDuty. We’re always looking to improve though, and will be continuing to add new features to this integration based on feedback going forward. 

If you’d like to give Sysdig Cloud a shot, please don’t hesitate to sign up for a free, 14 day trial above! If you’re already a PagerDuty user, you’ll hit the ground running.