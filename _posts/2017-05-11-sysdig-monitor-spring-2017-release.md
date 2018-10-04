---
ID: 4122
post_title: Sysdig Monitor spring 2017 release
author: Jorge Salamero Sanz
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-monitor-spring-2017-release/
published: true
post_date: 2017-05-11 13:00:12
---
We’re really excited to round up all the great functionality we’ve released so far this year on Sysdig Monitor into our Spring Release.

Here we’re going to give you a quick run-down on the most important features and where possible, link you to more detailed information on how to use the features.

If you’ve got questions on any of these features, or want our assistance using them, we’d love to help you!

Chat us in through the app or drop us a line at <a href="https://twitter.com/sysdig" target="_blank">@sysdig</a>.

## Updated integrations with orchestration systems

### Docker Swarm

We just released Docker 1.13 support together with out of the box support for Docker Swarm. New metrics like Swarm quorum, multiple new metadata labels like services, tasks, nodes, etc, and of course our topology maps now support Docker Swarm too. Read more about it on <a href="https://sysdigrp2rs.wpengine.com/blog/monitor-docker-swarm/" target="_blank">How to Monitor Docker Swarm</a>.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/docker_swarm_monitoring-1024x481.png" alt="Docker Swarm monitoring" width="1024" height="481" class="aligncenter size-large wp-image-4127" />][1] 
### Kubernetes 1.6

Kubernetes is a fast evolving project and we also move fast to provide support for the latest features available in version 1.6, including multi-cluster support for having all your clusters in your Sysdig Monitor account.

### Mesos improvements

Mesos, DC/OS and Mesosphere family are first class citizens at Sysdig Monitor. Now you don’t need to manually configure pulling out metadata and metrics from these platforms, it will be automatically detected by our agent.

## Usability improvements

Monitoring microservices is a complex task, getting visibility inside containers wasn’t trivial until the arrival of Sysdig. That’s why usability is important for us, and has been a main focus in the last months. Dashboard has received multiple improvements, simple changes that contribute to make monitoring with Sysdig a better experience:

*   Building a dashboard from any of the available templates now follows a more streamlined process thanks to the new wizard. This wizard shows the most popular templates, allows to filter them or select the initial scope. Remember that Sysdig Monitor provides dozens of templates for the different services we monitor and the orchestration tools we support.
*   If you would rather build your own dashboard from scratch, you can start from an empty one. Each dashboard can override every panel scope with a global dashboard scope.
*   You can share dashboards with other team members and copy them between teams (together with alerts), and if the new team needs to change the scope because they don’t have visibility permissions, they will be automatically notified.
*   Dashboard settings have been consolidated and panel edit controls improved, with an implicit save required to avoid erroneous changes.
*   A full screen mode is available for display mode in your ops war room.
*   Dashboards can be exported and imported through our API, find some examples in our <a href="https://github.com/draios/python-sdc-client/tree/master/examples" target="_blank">API Python wrapper</a>.

Some other improvements are available in other places like:

*   Comparisons operators have been implemented across the whole UI (dashboards panels and alerts too), being able to use the expressions ‘is’, ‘is not’, ‘not in’, ‘contains’, ‘does not contain’ and ‘starts with’.
*   Data tables show all available data with pagination, instead of the top 20 results, either if we have them as panels in a dashboard or in a Explore view. Columns can also be resized to the desired width.

## Improved alerting

We have improved how you can create an alert within Sysdig Monitor. Now you are presented with a wizard that allows to choose between:

*   Uptime: alerts to detect when a node, service/application or any other entity is down (not available or not defined).
*   Metric: alerts based on metrics applied across a given scope with trigger conditions like wait time or triggering multiple alerts for each instance of a given metadata.
*   Events: new functionality that allows count the number of events matching a criteria (names, tags, description), alert if the count exceeds a threshold. For example, alert if any of your pods is in a *CrashLoopBackOff* event.
*   Anomaly Detection: chose one of more metrics as a baseline and if the behaviour changes from what has been the pattern, will trigger an alert.
*   Group Outliner: think of a metric that should behave the same in multiple segments, e.g.: a service behind a load balancer or a set of pods inside a replicaSet. If any instance behaves atypically, you will get notified.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/alerts_wizard-1024x423.png" alt="" width="1024" height="423" class="aligncenter size-large wp-image-4124" />][2] 
## Performance and scale

A tremendous work has been happening behind the scenes. We have achieved a 10x scale in our backend, although we won’t stop there. This allows on premise customers to lower the hardware requirements for the same number of metrics and events they are collecting at the moment. Thanks to this improvement, SaaS customers have also seen their metrics and events limits increased!

From a user perspective we have released *Fast Dashboards*. Loading is now way faster, both dashboards as a whole and individual time series graphs.

But sometimes you just have too many metrics that you will never need, our agent can now filter them out with the `metrics_filter` option. This allows to include the metrics you know you want and exclude the ones you know you don’t want.

## Security improvements

A bunch of new features will make the most security concerned people very happy. Sysdig Monitor agent is now a <a href="https://blog.docker.com/2017/03/announcing-docker-certified/" target="_blank">Docker Certified container</a>, giving you all the guarantees that the container is using the best practices and is regularly scanned against known vulnerabilities. Actually, scanning all containers is now part of our CI process so either if you are using just our agent or deploying the Sysdig Monitor on-prem, you get updated images with latest fixes.

Improved authentication is all around too, Google oAuth for SaaS and on-prem, authentication between all the microservices that build up the on-prem version or you can restrict non-admin users from access to the access key, which can be rotated to accomplish with your credentials rotation policy.

## New time series charts

We got a new rendering for time series charts! The legend is now more interactive: you can hover over the lines or the legend entries to identify the metric, select and deselect them on the legend and sort them. Also, the legend will appear either on the side on wide panels or as a non-intrusive overlay when the panel is smaller.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/new_time_series_graphs-1024x212.png" alt="New time series graphs" width="1024" height="212" class="aligncenter size-large wp-image-4125" />][3] 
Some other improvements will be appreciated by the graphs fans. ‘null’ values are represented as no data instead of 0, you will be able to search for values not present in the chart, and multiple y-axis are supported. The scale can be either logarithmic and also fixed or dynamic depending on the values rendered.

## Percentiles

The agent can now calculate percentiles for the most common metrics. Just enable `percentiles: [50, 95, 99]` in your `dragent.yaml` and the new metrics will be available in the UI.

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/docker_swarm_monitoring.png
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/alerts_wizard.png
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/new_time_series_graphs.png