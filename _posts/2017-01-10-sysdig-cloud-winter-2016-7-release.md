---
ID: 3590
post_title: Sysdig Monitor winter 2016-7 release
author: Jorge Salamero Sanz
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-cloud-winter-2016-7-release/
published: true
post_date: 2017-01-10 06:08:09
---
Happy New Year 2017 to everyone! The cold hasn’t frozen our development pace, and during the last 3 months we have kept working hard on improvements to Sysdig Monitor. As is our tradition, we’re really excited to round up all the great stuff we’ve done into our Winter Release notes.

Let’s do a quick run-down on the most important features and where possible we will link you to deeper information on how to use the features.

## Container monitoring for teams

In November we announced Sysdig Teams to make easier monitoring containers and microservices in teams. Sysdig leverages Kubernetes (and other container orchestration tools) metadata to understand how your infrastructure is deployed; next step has been to **automatically define visibility and access control** based on the information that orchestration platform already has.

Users in multiple scenarios like running microservices apps, running PaaS for different teams within their organization, or with strong security requirements or data access restrictions are loving it! Read more about Sysdig Teams on <a href="https://sysdigrp2rs.wpengine.com/blog/introducing-sysdig-teams/" target="_blank">Introducing Sysdig teams – service-based access control for simpler and more secure Kubernetes & Docker monitoring</a> or checkout this 2 minute video for a quick intro:

<iframe width="560" height="315" src="https://www.youtube.com/embed/lkGtA_1IyXM" frameborder="0" allowfullscreen="allowfullscreen"></iframe>

## Dashboard panels scopes

This was a heavily requested feature: now each panel in any dashboard can **individually set their metrics scope**. Click on the gear icon on the panel and a configuration UI like in alerts will allow you to set the conditions that define the metrics included in the panel.

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/01/dashboard_scope_1.png" alt="Compare Prod vs Dev environments" width="1102" height="482" class="aligncenter size-full wp-image-3591" />

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/01/dashboard_scope_2.png" alt="Edit panel scope" width="953" height="254" class="aligncenter size-full wp-image-3592" />

## Making alerting easier

<a href="https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/204013259-Alerting-and-Notifications" target="_blank">Alert configuration interface</a> has been redesigned to make easier and more straightforward to configure alert parameters.

An updown process allows to define: the title, description, severity. The metric to alert on **Alert when**, the trigger conditions on **Aggregated across** and **Segment by** applies the conditions separately across each configured segment. There is a configurable wait time with **Over the last** and **Across** allows to specify if we want unique alerts for each segment or one for each.

As before you can send alerts to different <a href="https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/205375795-Setting-Up-Notification-Channels" target="_blank">notifications channels</a> and trigger Sysdig captures for out of band diagnosis and troubleshooting. **Captures default to 15 seconds** long now.

New accounts and new teams in Sysdig Monitor will see some **new alerts created by default** although disabled. These cover the basic operating system monitoring with alerts on metrics like disk, CPU, memory and IO.

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/01/create_alert.gif" alt="Create a Sysdig Monitor alert" width="610" height="648" class="aligncenter size-full wp-image-3593" />

## Custom App checks per container configuration

You can now define **custom <a href="https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/205147903-Metrics-integrations-Application-Checks" target="_blank">App checks</a> configuration for every container**. This allows to delegate monitoring configuration to the application teams. A variable called `SYSDIG_AGENT_CONF` will be checked for each monitored process, its parents and the Docker container. The value of this variable should follow the same syntax than on `dragent.yaml`, for example add to your Dockerfile or Pod definition:

<pre>ENV SYSDIG_AGENT_CONF {app_checks: [{ name: redisdb, pattern: {comm: redis-server}, conf: { host: 127.0.0.1, port: 6379, password: protected} }] }.
</pre>

This is available from Sysdig Monitor agent 0.43.0 so remember to keep your agents updated to the last version, <a href="https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/203572399-Sysdig-Cloud-Agent-Update-Uninstall" target="_blank">here is how to do it</a>.

## Improved metrics time traveling

<a href="https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/204889655-Time-Navigation" target="_blank">Time navigation</a> UI has been improved to make it easier to **jump back and forward in time**, selecting different graphing periods, from the last 10 seconds to 2 weeks time. Introducing exactly the date you want to visualize can be done before rendering the dashboard and old metrics gets consolidated and if the time selected needs to be slightly readjusted to match the consolidation, we will let you know in the UI.

## Some other features

### New metrics, MORE metrics, and metadata tags

There are a bunch of new metrics that you can monitor, and new tags to be used within your monitoring configuration like `container.image.id`. See the <a href="https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/204931155-Sysdig-Cloud-Metrics" target="_blank">full list here</a>.

File system usage metrics have been updated: `fs.root.used.percent` contains the usage of the root filesystem while `fs.largest.used.percent` is the usage of the largest file system, previously we had `fs.used.percent` that was the sum of all filesystems in use.

There are also new default limits on the metrics the agent sends to Sysdig Monitor: 300 beans per process, 500 metrics per process and 500 metrics per host.

### Rate aggregation available

A new <a href="https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/217873263-Aggregation-settings" target="_blank">rate aggregation</a> setting is available in addition to existing Average, Sum and Min/Max options. Rate returns the average value of the metric across the time period being evaluated. This is typically used in always increasing counter metrics like network traffic, file I/O, etc. This can be used both in graphs and alerts.

### Sysdig in Rancher Catalog

Open source Sysdig and Sysdig Monitor are now <a href="http://rancher.com/sysdig-announces-integrated-docker-monitoring-in-rancher/" target="_blank">both available in the Rancher Catalog</a>. The Rancher Catalog is an awesome service provided with the Rancher platform that lets you one-click deploy popular applications directly into your stack. So now, with the power of Sysdig + Rancher, in just a couple seconds you can have deep, container-native visibility and monitoring deployed across your entire infrastructure.

### Sysdig Monitor status page

We want you to always have the latest on Sysdig Monitor's uptime status, performance, and recent events. That’s why we launched a <a href="http://status.sysdigcloud.com/" target="_blank">status page</a> where we'll keep you continually updated. Remember you can also keep updated on product features through the <a href="https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/207928106-What-s-New" target="_blank">What’s New page</a> and <a href="https://sysdigrp2rs.wpengine.com/sysdig-cloud-changelogs/" target="_blank">RSS</a>.

### OpsGenie notification channel

We now have a native <a href="https://www.opsgenie.com/docs/integrations/sysdig-cloud-integration" target="_blank">alerting integration with OpsGenie</a>.

### Invoices in the App

Dealing with invoices is not a gratifying task, that’s why if you need the latest or any old invoices for your billing department you can get them a few clicks aways within the Sysdig Monitor app.

Have you tried any of these new features yet? We would love to hear from you on <a href="https://twitter.com/sysdig" target="_blank">@sysdig</a>, we love feedback and features request, help us to shape the future of container monitoring and troubleshooting!