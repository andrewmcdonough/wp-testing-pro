---
ID: 4977
post_title: 'Prometheus metrics: instrumenting your app with custom metrics and autodiscovery on Docker containers'
author: Mateo Burillo
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/prometheus-metrics/
published: true
post_date: 2017-10-18 04:30:57
---
Prometheus metrics allow you to instrument your app with tagged custom metrics. We will talk about the format, types, benefits, how to instrument an application with Prometheus libraries for custom metrics and how [Sysdig Monitor][1] autodiscovers these metrics on Docker containers in Kubernetes.

## What is Prometheus vs Prometheus metrics? {#what-is-prometheus-vs-prometheus-metrics-}

*… And Prometheus stole the metrics from Mount Olympus and gave them to mankind.*

[Prometheus][2] is an open-source monitoring solution, oriented towards highly dimensional data, decentralized and autonomous data sources with a powerful query language.

Prometheus metrics are increasingly becoming a defacto standard for custom metrics instrumentation.

Instrumenting your application allows you to measure things like:

*   How much time the customers spend on average on the product checkout page
*   How big is a certain data structure in memory
*   How many database requests are made by a certain code function

Prometheus certainly provides a lot of visibility to understand how things are working internally, but to do so requires instrumenting (modifying) your application code.

Example of blog post edit.

Instead of handling the metric calculation and exporting it on your own, the Prometheus monitoring project provides libraries for more than 10 programming languages that you can use to achieve getting application metrics. Don’t think of these libraries as specific for monitoring only with Prometheus, actually they can export metrics to <a href="https://github.com/prometheus/docs/blob/master/content/docs/instrumenting/exporters.md" target="_blank" rel="noopener">different time-series databases</a> like <a href="https://github.com/influxdata/influxdb" target="_blank" rel="noopener">InfluxDB</a>, <a href="http://opentsdb.net/" target="_blank" rel="noopener">OpenTSDB</a> or <a href="https://graphiteapp.org/" target="_blank" rel="noopener">Graphite</a>.

Sysdig Monitor is great at discovering and tagging custom metrics, and like we do with <a href="https://sysdigrp2rs.wpengine.com/blog/how-to-collect-statsd-metrics-in-containers/" target="_blank" rel="noopener">statsd</a> or <a href="https://sysdigrp2rs.wpengine.com/blog/gain-insights-jvms-sysdig-clouds-jmx-metrics/" target="_blank" rel="noopener">JMX</a>, **Sysdig automatically detects Prometheus metrics too!** [tweet_box design="default" float="none"]Sysdig Monitor automatically detects and imports #Prometheus metrics! Scale your monitoring now! Application Performance Monitoring[/tweet_box] 

## Prometheus metrics format: dot-metrics vs tagged metrics {#prometheus-metrics-format-dot-metrics-vs-tagged-metrics}

Let’s start with **dot-notated metrics**. for those of you familiar with the statsd metric format, this will be nothing new. In essence, everything you need to know about the metric is contained within the name of the metric. For example:

    production.server5.pod50.html.request.total
    production.server5.pod50.html.request.error

These metrics provide the detail and the hierarchy needed to effectively utilize your metrics. In order to make it fast and easy to use your metrics, this model of metrics exposition suggests that if you’d like a different aggregation, then you should calculate that metric up front and store it. So, with our example above, let’s suppose we were interested in request metrics across the entire service. We might flip our metrics to look something like this:

    production.service-nginx.html.request.total
    production.service-nginx.html.request.error

You could imagine many more combinations of metrics that might work for you. In fact, if you <a href="https://stackoverflow.com/questions/17542088/naming-pattern-in-graphite-and-statsd" target="_blank" rel="noopener">read a little</a> about this on sites like StackOverflow, you’ll see that the exact setup that you’d end up with is somewhere between art and science.

The Prometheus metric format takes a flat approach to naming metrics. Instead of a hierarchical, dot separated name, you have a name combined with a series of labels or tags:

`<metric name>{<label name>=<label value>, ...}`

A time series with the metric name *http_requests_total* and the labels *service="service"*, *server="pod50"* and *env=”production”* could be written like this:

`http_requests_total{service="service", server="pod50", env="production"}`

Highly dimensional data basically means that you can associate any number of application specific labels to every metric you submit. These labels are the key-value pairs that will be used for grouping / graphing / segmentation / computation of composite views.

Imagine a typical metric like `requests_per_second`, every one of your web servers is emitting these metrics. Now you add the labels (or dimensions):

*   Web Server software (Nginx, Apache)
*   Environment (production, staging)
*   HTTP method (POST, GET)
*   Error code (yes, no)
*   HTTP response code (number)
*   Endpoint (/webapp1, /webapp2)
*   Datacenter zone (east, west)

And voila! You already have N-dimensional data, and can easily obtain the following example graphs:

*   Total number of requests per web server in production
*   Number of HTTP errors using Apache for webapp2 in staging
*   Number of POST requests processed per datacenter in a certain zone

This comes at the cost of heavier data post-processing of course, but modern monitoring systems are prepared for that.

## When do you need to do code instrumentation? {#when-do-you-need-to-do-code-instrumentation-}

Gaining insights of your applications sounds like a fantastic idea, and it is! But like all things in life, it has a few trade-offs to consider:

*   Instrumenting with APM is simple code, but it is code nonetheless. This means that it needs to be done at development time. This adds more complexity, and potentially brings software bugs. Legacy or external applications are typically very hard to instrument. Sometimes if developers haven’t instrumented the application, it’s just out of scope for operations people to do it.

*   Performance overhead <span style="font-weight: 400;">–</span> no monitoring is free. If you plan to monitor critical, highly optimized loop code you may end up using more time emitting metrics than doing the actual work. More metrics in your monitoring backend will also require more computing resources there.

## Types of Prometheus metrics {#types-of-prometheus-metrics}

The Prometheus client libraries offer <a href="https://prometheus.io/docs/concepts/metric_types/" target="_blank" rel="noopener">four core metrics types</a>:

*   **Counter:** A cumulative metric that represents a single numerical value that only ever goes up. There is a counter example in the code above.

*   **Gauge:** A gauge is a metric that represents a single numerical value that can arbitrarily go up and down. For example, the number of active connections.

*   **Histogram:** A histogram samples observations (usually things like request durations or response sizes) and counts them in configurable buckets. It also provides a sum of all observed values. Most commonplace metrics fall under this category.

*   **Summary:** Similar to a histogram, a summary samples observations (usually things like request durations and response sizes). While it also provides a total count of observations and a sum of all observed values, it calculates configurable quantiles over a sliding time window. A typical example is getting the 95th percentile of requests duration to understand the worst case responsiveness of our system.

A code example could be useful here. If you take a look at the `requests_per_second` N-dimensional metric we just mentioned, you can come to the conclusion that… well, Sysdig Monitor can see inside your containers and processes, decode most popular application protocols like HTTP and different databases like MySQL/PostgreSQL, Redis, Mongo, Cassandra, etc., so you don’t really need Prometheus metrics instrumentation for monitoring requests, response time or errors ;-) (read more on must-have application key metrics on <a href="http://www.brendangregg.com/usemethod.html" target="_blank" rel="noopener">Brendan Gregg’s USE method</a>).

But what if you need a custom metric or some other value that you can only obtain from the source code of your app? In these cases is where Prometheus metrics shine.

## How to add custom metrics to your application {#how-to-add-custom-metrics-to-your-application}

Let’s see a real small and quick example using Python.

First, if you haven’t done it already, install the python Prometheus library:

`sudo pip install prometheus_client`

And execute this trivial python script (but enough to see metric labels and histograms in action)

<script src="https://gist.github.com/mateobur/4bfb70bbd463e2a5f56b82e6a44a2063.js"></script> You can just display the raw Prometheus data accessing `http://localhost:9100`: 
    ....
    # HELP python_info Python platform information
    # TYPE python_info gauge
    python_info{implementation="CPython",major="2",minor="7",patchlevel="13",version="2.7.13"} 1.0
    # HELP function_exec_time Time spent processing a function
    # TYPE function_exec_time histogram
    function_exec_time_bucket{func_name="func1",le="0.005"} 0.0
    function_exec_time_bucket{func_name="func1",le="0.01"} 0.0
    ....
    

## Why does Sysdig love Prometheus metric support? {#why-is-sysdig-loves-prometheus-metric-support-}

Prometheus libraries are open source and make pushing out metrics from any software platform and/or programming language super easy. As a result, a lot of the tools we love and use are already bundling it and you can enable application-specific metrics with little configuration.

Take <a href="https://sysdigrp2rs.wpengine.com/blog/monitor-linkerd/" target="_blank" rel="noopener">LinkerD</a> as an example, you can enable Prometheus metrics with just <a href="https://linkerd.io/administration/telemetry/" target="_blank" rel="noopener">one configuration line</a>.

Even if you already have a full-blown Prometheus-based monitoring on your Kubernetes cluster, you can integrate now Sysdig Monitor without friction. Sysdig will autodiscover the metrics you already expose and provide additional metadata on top, the same way we tag absolutely everything. All this without instrumenting again your code and systems nor cause any incompatibility or disturbance to your current setup.

## Look Ma, no queries! Visualizing Prometheus metrics in Sysdig Monitor {#look-ma-no-queries-visualizing-prometheus-metrics-in-sysdig-monitor}

We’ve applied all the goodness of the Sysdig Explore functionality (in addition to dashboards, alerting etc…) to your Prometheus metrics. So, in one screenshot:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/10/sysdig_interface_prometheus.png" alt="prometheus metrics interface sysdig" width="2559" height="1291" class="alignleft size-full wp-image-5050" />][3] 
The *Explore* interface lets you use all your labels to orient the way you display your metrics. That means you can take a physical view (host, datacenter, etc) , or with a click, change to have a logical view (Kubernetes deployments, pods, etc). Clicking on any row then scopes your metrics to that subsection of your infrastructure.

You can then browse or search your metrics, click on one of them, and then adjust aggregations and segment the metric by other, logical breakouts.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/10/sysdig_prometheus_metrics_list.png" alt="prometheus metrics list sysdig" width="417" height="409" class="alignleft size-full wp-image-5049" />][4]   
  
And just like that, you’ve got all of your data at your fingertips with a few clicks. What’s more, you can easily enable your entire team to use this data, without trying to teach them a new DSL or (worse) being their query monkey.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/10/sysdig_prometheus_metric_graph.png" alt="prometheus metric graph sysdig" width="2460" height="796" class="alignleft size-full wp-image-5048" />][5]   
  
On top of that, you can use Prometheus metrics and dashboards to configure the related <a href="https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/204013259-Alerting-and-Notifications" target="_blank" rel="noopener">alerts and notifications</a>:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/10/prometheus_metrics_alert-1.png" alt="Prometheus metrics alert" width="1297" height="589" class="alignleft size-full wp-image-5052" />][6]   
  
Willing to try this on your system already? This <a href="https://sysdigdocs.atlassian.net/wiki/spaces/Monitor/pages/204603650" target="_blank" rel="noopener">how to enable and configure Prometheus metrics on Sysdig agent</a>.

## Prometheus exporters and side-car containers {#prometheus-exporters-and-side-car-containers}

Much of the popular server software already bundles its own stats and metrics <span style="font-weight: 400;">–</span> think of the Apache status page, for instance. <a href="https://prometheus.io/docs/instrumenting/exporters/" target="_blank" rel="noopener">Exporters</a> collect and adapt these metrics to be consumed by Prometheus. You can find exporters as a single process or as a side-car container to a Kubernetes pod.

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/10/prometheus_exporter-1024x503.png" alt="prometheus exporter" width="1024" height="503" class="aligncenter size-large wp-image-5045" />  
  
In these cases, the Sysdig agent automatically collects the exporter data for you <span style="font-weight: 400;">–</span> although we recommend using Sysdig’s application-specific checks as they typically offer a better experience (more visibility and tagged metrics).

## Conclusions {#conclusions}

We have seen how to extend container and service visibility with Prometheus custom metrics for APM-style monitoring and to gain visibility into what your code it's doing. Prometheus libraries are broadly used for custom metrics, no matter which <a href="https://sysdigrp2rs.wpengine.com/blog/monitoring-kubernetes-with-sysdig-cloud/" target="_blank" rel="noopener">Kubernetes monitoring</a> system you use.

Sysdig Monitor will now unlock all the goodness from Prometheus monitoring with no disruption to your developers, and no work on maintaining or building a skill set around a complex new tool. With compatibility to co-exist or migrate <span style="font-weight: 400;">– </span>Prometheus metrics and Sysdig are best friends.

Check out our <a href="https://github.com/bencer/example-voting-app" target="_blank" rel="noopener">code-instrumented version</a> of the Docker’s demo example-voting-app and test it yourself with a 15 host, 300 container <a href="https://sysdigrp2rs.wpengine.com/" target="_blank" rel="noopener">Sysdig Monitor free trial!</a>

 [1]: https://sysdigrp2rs.wpengine.com/product/monitor/
 [2]: https://prometheus.io/
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/10/sysdig_interface_prometheus.png
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/10/sysdig_prometheus_metrics_list.png
 [5]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/10/sysdig_prometheus_metric_graph.png
 [6]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/10/prometheus_metrics_alert-1.png
