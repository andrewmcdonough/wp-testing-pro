---
ID: 7568
post_title: Sysdig Monitor summer 2018 release.
author: Eric Carter
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-monitor-summer-2018-release/
published: true
post_date: 2018-06-21 09:45:30
---
It’s the first day of summer and the perfect opportunity for our summer Sysdig Monitor release round up. For those of you following our progress, we use these blogs to showcase the work we’ve done to add increased functionality, scale, and usability with Sysdig Monitor.

What follows are quick descriptions of all the good stuff we’ve made available over the past few months. This includes a lot of enhancements to look-and-feel, dashboards, metrics, Kubernetes and Prometheus support and more.   
  
Got questions? Need help? Let us know through the app, via [Slack][1], or drop us a line at [@sysdig][2].

## Kubernetes-related features {#kubernetesrelatedfeatures}

### Kubernetes Node Ready alert {#kubernetesnodereadyalert}

A new out-of-the-box alert is available to provide notifications when a Kubernetes node is *not* ready. By default the alert is set to warning, but severity is user configurable. This feature will help you make sure you have the resources you need up and ready for your Kubernetes-based services.

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/node-not-ready.png" alt="kubernetes node ready" width="2698" height="1118" class="alignnone size-full wp-image-7571" />

### Revised alerting with Kubernetes metrics {#revisedalertingwithkubernetesmetrics}

Alert configuration settings for Kubernetes metrics now limit scope and segmentation based on the metric that is selected to allow for more accurate alerting. Check out [our support page ][3]for more details.

## Prometheus-related features {#prometheusrelatedfeatures}

### Support for Prometheus histogram metrics {#supportforprometheushistogrammetrics}

Sysdig Monitor can now ingest a [Prometheus histogram metric type][4] and visualize them in a chart to show the distribution of specific metrics. This gives Prometheus users even more visibility options within Sysdig Monitor to help provide a greater level of insight.

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/prometheus-histogram-monitor.png" alt="sysdig prometheus histograms" width="861" height="343" class="alignnone size-full wp-image-7573" />

### Default Istio dashboards {#defaultistiodashboards}

Sysdig provides out of the box dashboards for monitoring Istio using Prometheus exporters. Istio, a<span>n open platform to connect, manage, and secure microservices, is growing in popularity - especially with Kubernetes users. Now you get instant visibility to the most important metrics for this service mesh capability.</span>

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/istio-dashboards.gif" alt="Sysdig Istio dashboards" width="1276" height="759" class="alignnone size-full wp-image-7574" />

### Link to Grafana plugin {#linktografanaplugin}

Did you know you can add Sysdig as a Grafana data source? To help you get started visualizing Sysdig-collected metrics in Grafana, we've added a Grafana Plugin link to the help menu that takes you to the <a href="https://github.com/draios/grafana-sysdig-datasource" target="_blank" rel="noopener">setup instructions</a>.

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/grafana-plugin-monitor.png" alt="sysdig monitor grafana link" width="329" height="259" class="alignnone size-full wp-image-7575" />

[tweet_box design="default" float="none"]Check out all the cool stuff @sysdig has released with Sysdig Monitor - #kubernetes #prometheus and more![/tweet_box] 

## Usability-related features {#usabilityrelatedfeatures}

### New UI design {#newuidesign}

Our new user interface provides a more modern framework for interacting with the product. Navigation is re-oriented from a top-of-screen menu to an icon-driven left side panel, providing more space for viewing your metrics and dashboards. [Click here for a quick video introduction!][5]

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/new-ui.gif" alt="new sysdig user interface" width="1441" height="779" class="alignnone size-full wp-image-7576" />

### Spotlight {#spotlight}

Want a simple way to quickly see what matters most in your environment? Spotlight helps you quickly discover, detect, and optimize your infrastructure and services. A Spotlight health check shows you new integrations, infrastructure, app, and agent status, and more at-a-glance.

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/sysdig-spotlight.png" alt="sysdig spotlight" width="1920" height="1080" class="alignnone size-full wp-image-7577" />

### New Explore design {#newexploredesign} We've redesigned Sysdig Monitor's Explore page to give you extra screen space to view your killer dashboards and metrics. The new vertical layout helps you see more and get to what you need faster. 

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/Explore-redesign-1.gif" alt="sysdig explore redesign" width="1649" height="955" class="alignnone size-full wp-image-7579" />

### Icon labels {#iconlabels}

New icon labels appear on hover to clarify underlying function for users. This is a part of our ongoing efforts to increase simplicity and usability across the Sysdig platform.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/icon-labels-1.gif" alt="sysdig icon labels" width="880" height="763" class="alignnone wp-image-7649 size-full" />][6]

### Redesigned login screen {#redesignedloginscreen}

We've put a new, more modern face on the Sysdig Monitor login screen.

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/login-redesign-sysdig-monitor.png" alt="sysdig login redesign" width="617" height="414" class="alignnone  wp-image-7581" />

### Text panels {#textpanels}

You can now add text panels to your dashboards to provide additional information. Text panels can be used as title headers or to provide additional context that you would like to communicate. Features limited [markdown support][7].

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/text-panels.gif" alt="sysdig text panels" width="1256" height="710" class="alignnone size-full wp-image-7582" />

### Default entry point {#defaultentrypoint}

Admins can now set a default entry point for a team to simplify the onboarding process. This determines the first page users see when they start the application (e.g., a specific dashboard, settings, etc.).

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/sysdig-default-entry-point.png" alt="sysdig default entry point" width="1197" height="671" class="alignnone size-full wp-image-7583" />

### Copy and share groupings {#copyandsharegroupings}

Now you can copy and share unique groupings with all of your teams. This helps save times when working across groups and helping team members see critical metrics and more.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/copy-share-groupings-1.gif" alt="sysdig copy share groupings" width="1176" height="763" class="alignnone wp-image-7648 size-full" p="" />][8] 

### Test notification channels {#testnotificationchannels}



New test function lets you pre-test your notification channels like, email, Slack, PagerDuty, etc. This capability helps give you the assurance that all messages generated by Sysdig Monitor will be received.



<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/test-notifications.gif" alt="sysdig test notifications" width="1276" height="736" class="alignnone size-full wp-image-7585" />



### Resizable columns {#resizablecolumns}



The UI now allows columns to be resized for all tables in the application including alerts, events, teams, and users.



### Suggest mode {#suggestmode}



Suggest mode auto-selects only the relevant dashboards and metrics, hiding any inapplicable views. This is now the normal mode of operation. The turn on/off option is no longer available.



### Additional UI updates {#additionaluiupdates}



We've simplified the dashboard panel copy function and added a duplicate panel option in menu. We've also redesigned the dropdowns in the top right header including making it easier to quickly see and select your teams.



## Metric-related features {#metricrelatedfeatures}



### Golden Signals dashboards {#goldensignalsdashboards}



New Service Golden Signal dashboards provide out-of-the-box metrics that developers need when launching and monitoring a service or app. Includes slowest transactions, latency, request volume, error rates, and most requested URLs.



<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/service-golden-signals.png" alt="sysdig golden signals" width="1920" height="1080" class="alignnone size-full wp-image-7586" />



### Multiple segments for a single metric {#multiplesegmentsforasinglemetric}



You can now add up to five different segments for a given metric in time-series and stacked area panels.



<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/single-metric-multiple-segments.png" alt="scoping multiple segments" width="480" height="400" class="alignnone size-full wp-image-7587" />



### Alert on rate of change {#alertonrateofchange}



Introducing a new 'rate of change' math function for metrics. Now you can alert by the rate at which a metric changes vs. a static threshold. For example, a default alert: *Rate of change of disk usage* alerts you if your disk usage increases more than x% in a day.



<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/rate-of-change-disk-sysdig.png" alt="sysdig rate of change" width="582" height="426" class="alignnone size-full wp-image-7588" />



### 'Compare to' for number panels {#comparetofornumberpanels}



Metric number panels now feature a configurable 'Compare to' function to display the change in measurement since a previous time frame. Provides insight into the increase or decrease of metrics over time.



<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/compare-to-number-panel.png" alt="sysdig compare to panels" width="1920" height="1080" class="alignnone size-full wp-image-7589" p="" /> 

### 'Compare to' for timeseries {#comparetofortimeseries}



In your timeseries line charts you can now compare time-shifting metrics to easily spot trends and anomalies. With compare-to for timeseries you can configure and observe how one or more metrics have changed since a previous time (e.g., 1 hour ago or 2 days ago).



<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/compare-to-timeseries-monitor.png" alt="sysdig compare to timeseries" width="1036" height="333" class="alignnone size-full wp-image-7591" />



### Export table data as JSON/CSV {#exporttabledataasjsoncsv}



You can now download table data in JSON or CSV format for offline viewing and analysis. This is a great way to share and work with data offline.



<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/export_csv-json.png" alt="sysdig export csv json" width="1920" height="1080" class="alignnone size-full wp-image-7592" />



### New metrics for cpu core usage {#newmetricsforcpucoreusage}



We've added *cpu.cores.used* and *cpu.cores.used.percent* that align with the way Kubernetes exposes cpu usage. Now you can compare values using kube-state-metrics such as kubernetes.node.capacity.cpuCores, kubernetes.pod.resourceLimits.cpuCores in order to determine if resources are oversubscribed. These metrics are also key for capacity planning and chargeback calculations.



<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/CPU-metrics-monitor.png" alt="sysdig cpu" width="585" height="389" class="alignnone  wp-image-7593" />



### Improved documentation for CPU metrics {#improveddocumentationforcpumetrics}



Our [Sysdig Monitor Metrics support page][9] now features updated CPU metrics descriptions to provide more insight into each available metric.  
  




**Interested in seeing more?** Check out our new [walkthrough video][10] for a quick tour of Sysdig Monitor.

 [1]: https://slack.sysdigrp2rs.wpengine.com/
 [2]: https://twitter.com/sysdig
 [3]: https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/360000204446
 [4]: https://prometheus.io/docs/concepts/metric_types/#histogram
 [5]: https://youtu.be/flIblk5mQys
 [6]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/icon-labels-1.gif
 [7]: https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/360004217792
 [8]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/copy-share-groupings-1.gif
 [9]: https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/204931155
 [10]: https://youtu.be/HuLp-xBtJJs