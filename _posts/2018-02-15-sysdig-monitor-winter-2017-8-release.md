---
ID: 5961
post_title: Sysdig Monitor winter 2017-18 release
author: Eric Carter
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-monitor-winter-2017-8-release/
published: true
post_date: 2018-02-15 13:49:26
---
We’ve been busy since our fall <a href="https://sysdigrp2rs.wpengine.com/blog/sysdig-monitor-fall-2017-release/" target="_blank" rel="noopener">Sysdig Monitor release</a>. A lot of great features have been rolled out and our most recent update delivers some really important integrations and capabilities. Things are moving quite fast in the container infrastructure space and we are all about continuous improvement – from the user experience to new metrics and new ways to visualize your environment.

Read on to get a quick overview of what’s new. If you have any questions or want our assistance with anything you see here, we’d love to help! Chat us through the app, via <a href="https://slack.sysdigrp2rs.wpengine.com/" target="_blank" rel="noopener">Slack</a>, or drop us a line at <a href="https://twitter.com/sysdig" target="_blank" rel="noopener">@sysdig</a>.

## kube-state-metrics {#kubestatemetrics}

Sysdig Monitor now collects <a href="https://github.com/kubernetes/kube-state-metrics" target="_blank" rel="noopener">kube-state-metrics</a> for monitoring and alerting on the state of Kubernetes objects. New kube-state-metrics dashboards provide out-of-the-box visibility of a wide range of metrics for nodes, namespaces, services, daemonSets, jobs, replicaSets and pods. *Note: requires **<a href="https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/203572399" target="_blank" rel="noopener">update to the Sysdig agen</a>*t* version 0.77.0 or higher.* <a href="https://sysdigrp2rs.wpengine.com/blog/introducing-kube-state-metrics/" target="_blank" rel="noopener">Read more on our blog here.</a>

<a href="https://youtu.be/P40QTxNk0j8" target="_blank" rel="noopener"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/kube-state-metrics.png" alt="kube-state-metrics" width="1348" height="737" class="alignnone wp-image-6195 size-full" /></a>

## Public URL dashboards {#publicurldashboards}

Ever want to share a killer dashboard with a colleague who is not a Sysdig Monitor user? Now you can! Just pick, click, and send your URL.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/public-url-dashboards.gif" alt="public URL dashboards" width="992" height="568" class="alignnone size-full wp-image-6197" />][1]

## [tweet_box design="default" float="none"]Check out all the new Sysdig Monitor goodness with #Kubernetes kube-state-metrics, public dashboards and more![/tweet_box] {#tweetcheckoutallthenewsysdigmonitorstufffromsysdigwithkuberneteskubestatemetricspublicdashboardsandmoretweet}

## 

## Explore grouping and scoping enhancements {#teammanagerrole}

We’ve massively simplified grouping and scopes. Our new approach gives you better, more precise data - with less chance of invalid groupings (e.g. Kubernetes deployment > hostname). Have questions? Contact Customer Success and we’ll analyze your account for you!  
  
[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/kubernetes-groupings-2.gif" alt="logical kubernetes groupings" width="964" height="528" class="alignnone size-full wp-image-6199" />][2]

## Team Manager role

We’ve introduced a new 'Team Manager' role that provides the privilege to add, delete, and modify team users as well as grant read or edit access.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/team-manager-role.png" alt="team manager role" width="181" height="153" class="alignnone wp-image-6198 size-full" />][3]

## Read-only users {#readonlyusers}

The addition of a read-only role in Sysdig Monitor enables you to define 'Read user' or 'Edit user' roles for each team to which a user belongs.

*   A read user can only use Sysdig Monitor in read-only mode, with no permission to create, edit, or delete dashboards, alerts, etc.
*   An edit user is allowed to make changes. This is a per team role defined by Admin users.



[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/readuser.png" alt="read-only user" width="1349" height="646" class="alignnone size-full wp-image-6202" />][4]

## Inspect captures with Sysdig Inspect {#inspectcaptureswithsysdiginspect}

We released <a href="https://sysdigrp2rs.wpengine.com/blog/sysdig-inspect/" target="_blank" rel="noopener">Sysdig Inspect</a> back in September: a powerful interface for container troubleshooting from captures files. Sysdig Inspect has been built on top of Sysdig open source and is also available for Mac and Linux users. Now you can open capture files directly from your browser when <a href="https://sysdigrp2rs.wpengine.com/blog/new-integrated-troubleshooting-sysdig-monitor/" target="_blank" rel="noopener">using Sysdig Monitor</a>.

## [<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/01/open_with_sysdig_inspect.gif" alt="Sysdig Inspect" width="2880" height="1746" class="alignnone size-full wp-image-5987" />][5] {#imagealttextimage_4gif}

## New metrics, dashboards and visualizations {#newmetricsdashboardsandvisualizations}

### Support for JMX metrics from Java 9 {#supportforjmxmetricsfromjava9}

<p id="sysdigmonitornowsupportsjmxmonitoringforjava9applicationstoenablecollectionofjava9metricsupdatetothelatestsysdigagentformoredetailsreviewthesysdigagentchangelogs">
  Sysdig Monitor now supports <a href="https://sysdigrp2rs.wpengine.com/blog/gain-insights-jvms-sysdig-clouds-jmx-metrics/" target="_blank" rel="noopener">JMX monitoring</a> for Java 9 applications. To enable collection of Java 9 metrics, update to the latest Sysdig Agent. For more details, review the <a href="https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/115002168946" target="_blank" rel="noopener">Sysdig Agent changelogs</a>.
</p>

### Memcached default dashboard {#memcacheddefaultdashboard}

A new default dashboard has been added to the Sysdig Monitor Explore page where you can see the most important <a href="http://memcached.org/" target="_blank" rel="noopener">Memcached</a> performance monitoring metrics including connections, commands, get hits/misses, and evictions.

### New panel type: Histograms {#newpaneltypehistograms}

With histograms you can see the distribution of values for a metric over hosts, containers, etc. A typical use case for histograms is to detect outliers. For example, you can plot a histogram of response time by containers to find out which response time ranges are the most common and which containers are outside the normal range. Along the x-axis are buckets/ranges of the metric value. The bars represent how many segments are in each range. By hovering over with your cursor, you can see the names of the segments in each range.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/histograms.png" alt="sysdig monitor histograms" width="1381" height="620" class="alignnone size-full wp-image-6205" />][6]

### AWS ALB metrics support {#awsalbmetricssupport}

<a href="https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html" target="_blank" rel="noopener">AWS Application Load Balancer</a> (ALB) metrics are now supported with an entry in our newly introduced data source dropdown menu, a new explore table configuration, and a new default dashboard that includes metrics like healthy and unhealthy hosts, requests per second, response time, and HTTP response by codes (2XX, 3XX, 4XX and 5XX).

### Export time series data as JSON/CSV {#exporttimeseriesdataasjsoncsv}

We added two new options to time series and stacked area charts that enable you to download raw data in JSON or CSV format to run custom data analysis.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/export_csv-json.png" alt="export as csv json" width="1046" height="257" class="alignnone size-full wp-image-6206" />][7]

## Enhanced workflows, user experience, and help info {#enhancedworkflowsuserexperienceandhelpinfo}

### New time navigation {#newtimenavigation}

Time navigation is now located at the bottom of the Explore and Dashboards page making it easy to click to a specific time frame. Custom time ranges now have a better UX and zoom in/out controls are more prominent.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/time-nav-2.png" alt="time navigation" width="840" height="231" class="alignnone size-full wp-image-6209" />][8]

### Time and group metric aggregation suggestions {#timeandgroupmetricaggregationsuggestions}

Confused about which time and group aggregation is used on each metric? Now you can have access to a quick recap with the best practices on how to aggregate metrics based on type (percentage, latency, etc.).

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/aggregation_recap.gif" alt="time aggregation" width="1209" height="675" class="alignnone size-full wp-image-6208" />][9]

### Panel metric information {#panelmetricinformation}

Metric information (metric name with time and group aggregation) can be seen when hovering over a panel (bar charts, time series, stacked and number panels). This is helpful to give more context about the purpose of the panel.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/01/panel_metric_information.png" alt="panel metric info" width="555" height="171" class="alignnone size-full wp-image-5986" />][10]

### Improved Explore event workflow {#improvedexploreeventworkflow}

Scope matching when exploring events has been improved, bringing you back into the right point in time, grouping and scope. Additionally, the Explore page has an ad-hoc single event mode showing you only the specific event that you are exploring.

### Suggest mode enabled by default {#suggestmodeenabledbydefault}

Last year we introduced suggest mode – available in 'Settings>Sysdig Labs' – as a way to boost your efficiency by showing only the views, metrics, and grouping presets applicable to your environment. This option has proven so popular that it is now enabled by default.

### Rename of Admin team to Monitor Operations {#renameofadminteamtomonitoroperations}

As part of the broader Sysdig Platform initiative, 'Admin Team' within Sysdig Monitor is now renamed to 'Monitor Operations.' The Monitor Operations team will continue to behave the same as the previous Admin team:

*   The Monitor Operations team cannot be deleted.
*   Monitor Operations users have full visibility to all resources.
*   To change settings for any team, admins must switch to the Monitor Operations team. 



[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/monitor-operations.png" alt="sysdig monitor operations" width="438" height="259" class="alignnone size-full wp-image-6211" />][11]

### Custom headers for webhooks {#customheadersforwebhooks}

When using webhooks, typically used to pass authentication credentials, you can now add custom headers to pass along additional details with an outgoing request.

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/public-url-dashboards.gif
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/kubernetes-groupings-2.gif
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/team-manager-role.png
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/readuser.png
 [5]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/01/open_with_sysdig_inspect.gif
 [6]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/histograms.png
 [7]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/export_csv-json.png
 [8]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/time-nav-2.png
 [9]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/aggregation_recap.gif
 [10]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/01/panel_metric_information.png
 [11]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/monitor-operations.png