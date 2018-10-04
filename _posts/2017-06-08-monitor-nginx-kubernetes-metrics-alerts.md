---
ID: 4223
post_title: 'How to monitor Nginx on Kubernetes: Metrics alerts'
author: Mateo Burillo
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/monitor-nginx-kubernetes-metrics-alerts/
published: true
post_date: 2017-06-08 04:24:10
---
In this article we are going to show how to set up Nginx alerts associated with common failure points and early risk mitigation when running on Kubernetes. This is the second part of the <a href="https://sysdigrp2rs.wpengine.com/blog/monitor-nginx-kubernetes/" target="_blank"> How to monitor Nginx on Kubernetes </a> post. 

## How to monitor Nginx  {#how-to-monitor-nginx}

On the first part of these Nginx monitoring posts we saw the Nginx `stub_stats` module configuration, the relevant metrics (active connections, requests per second, etc) and how to work with them using Kubernetes metadata labels, illustrated with several examples: 

*   <a href="https://sysdigrp2rs.wpengine.com/blog/monitor-nginx-kubernetes/#nginx-stub_status-configuration-on-kubernetes" target="_blank"> Nginx stub_status configuration </a> in the Nginx Docker image running as Kubernetes pod using a ConfigMap. 
*   <a href="https://sysdigrp2rs.wpengine.com/blog/monitor-nginx-kubernetes/#monitoring-nginx-metrics" target="_blank"> Monitoring Nginx metrics </a> from service level metrics like requests per second to HTTP protocol metrics status code and service response time. 
*   <a href="https://sysdigrp2rs.wpengine.com/blog/monitor-nginx-kubernetes/#nginx-dashboards" target="_blank"> Nginx Dashboards </a> that give you an overview of your Nginx health and performance. 
*   <a href="https://sysdigrp2rs.wpengine.com/blog/monitor-nginx-kubernetes/#using-kubernetes-labels-in-graphs-to-monitor-nginx" target="_blank"> Using Kubernetes labels </a> to monitor Nginx. Several monitoring and troubleshooting use cases and examples applying the context-aware Kubernetes labels. 

Now that we have laid the foundations to monitor a Nginx deployment on Kubernetes, let’s configure the some of the most relevant alerts. 

## Nginx alerting  {#nginx-alerting}

Taking the Nginx deployments described in the last post as a starting point, we have to consider the most critical and/or most frequent failure scenarios. This is our checklist when configuring the most important alerts in a typical Nginx deployment on Kubernetes: 

*   [Kubernetes failing to deploy or run the Nginx service.][1] 
*   [Nginx image failing: PodCrashLoopBackOff?][2] 
*   [Nginx is dropping user connections.][3] 
*   [Degraded service quality / response time.][4] 
*   [Web backends reporting HTTP error codes.][5] 

<span>[tweet_box design="default" float="none"]Running #Nginx on #Kubernetes? These are the 5 alerts that you should setup! [/tweet_box]</span> 

### Alert on Nginx Kubernetes deployment  {#alert-on-nginx-kubernetes-deployment}

Probably the most urgent and obvious case would be the loss or unavailability of your Nginx pods in the Kubernetes cluster. 

Let’s configure this basic alert: 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/06/image1.png" alt="nginx service down" width="895" height="560" class="alignleft size-full wp-image-4226" />][6] 

<sub>Average of <code>kubernetes.replicaSet.replicas.running</code> over the last 2 minutes.</sub> 

You may be wondering *Why a threshold of 0.8?*. Take into account that this is an average value over a fixed time slot and that we want to alert as soon as possible. For this metric to be exactly 0 it needs to be 0 during the entire period. 

Typically, you will specify the number of Nginx replicas that you want to keep running using a <a href="https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/" target="_blank"> ReplicationController </a> or <a href="https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/" target="_blank"> ReplicaSet</a>. Kubernetes will spawn new nodes when required to meet your criteria. With this alert we want to check that absolutely none of our Nginx pods are running. 

What if Kubernetes fails to deploy new Pods but existing ones keep running? This might happen on a rolling deployment or when having auto-scaling. Then you need to also compare the current *running* and *desired* parameters. You can do that using an advanced alert like this one: 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/06/Selection_135.png" alt="ReplicaSet degraded" width="893" height="572" class="alignleft size-full wp-image-4228" />][7] 

<sub>Comparing <code>timeAvg(kubernetes.replicaSet.replicas.running) &lt; timeAvg(kubernetes.replicaSet.replicas.desired)</code> for the last 5 minutes.</code></sub>

If you want to know more about how to configure advanced alerts in Sysdig Monitor using time aggregation and optionally group aggregation, advanced alert syntax is described <a href="https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/208284466" target="_blank"> here</a>. 

### Nginx image failing to run  {#nginx-image-failing}

There are several reasons why the Nginx image deployment may not be able to run. Maybe the most common one is misconfiguration (errors in the config files of the pod services) but there are other cases like unmet image requirements or capacity limits. 

If you happen to deploy a container with any of these issues, it will fail and be restarted by Kubernetes in a loop. This is called *CrashLoopBackOff*. You want to be alerted of this failed deployment as soon as possible: 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/06/image3.png" alt="Crashloop" width="894" height="600" class="alignleft size-full wp-image-4229" />][8] 

<sub>Average of <code>kubernetes.pod.restart.count</code> higher than 3 for the last 2 minutes, <code>production</code> Kubernetes namespace.</sub>

Another way to detect problems with your image deployment is using the Kubernetes and Docker event alerts. 

In Sysdig Monitor you can see these events as circles added on top of any time series graph: 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/06/image4.png" alt="events" width="1121" height="702" class="alignleft size-full wp-image-4230" />][9] 

Hovering your mouse cursor over the event circle will show you a more detailed description. 

You can also create an event-based alert using the *Container killed* filter since it will happen repeatedly if Kubernetes is not able to bootstrap the image: 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/06/image5.png" alt="Container killed" width="893" height="644" class="alignleft size-full wp-image-4231" />][10] 

<sub>Kubernetes event contains the "Container killed" string, this event happens at least 5 times in the last 2 minutes.</sub>

Take into account that a single *Container killed* may be harmless or even expected depending on your deployment (for example Kubernetes rescheduling the pod in a different node). That's why we alert when the ratio of containers killed per minute is unusual.

#### Troubleshooting Nginx CrashLoopBackOff  {#troubleshooting-nginx-crashloopbackoff}

This scenario is one of the multiple cases when the <a href="https://sysdigrp2rs.wpengine.com/blog/troubleshooting-containers-when-theyre-gone/" target="_blank"> Sysdig Capture </a> in Sysdig Monitor together with <a href="https://www.sysdig.org/" target="_blank">Sysdig open-source</a> shows its potential to debug the underlying issues. Even after the containers that originated that error are gone and everything is back to normal again. 

You can enable it configuring these parameters on section 3 of the alert configuration form: 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/06/image6.png" alt="enable sysdig capture" width="893" height="272" class="alignleft size-full wp-image-4232" />][11] 

And retrieve the captures later on from the *Capture* tab in Sysdig Monitor interface. 

Using the `sysdig` and `csysdig` tools you can then inspect the system calls capture file. 

For example, listing the containers present in the capture file: 

    $ sysdig -r crashloop.scap -c lscontainers

The replica declares only 3 concurrent pods, but there are a lot of short-lived Nginx pods (see the <a href="https://gist.github.com/mateobur/fcf89a06b05ffc005aa2cda15f6c7f7d" target="_blank"> full output here</a>). 

Can you get the standard error output of a process that doesn’t exist anymore? Using Sysdig, you can! 

    $ sysdig -r crashloop.scap -A -c stderr proc.name=nginx
        
        nginx: [emerg] invalid parameter “servre” in /etc/nginx/nginx.conf:71

Let’s use the visual `csysdig` ncurses tool now that we have a pretty good clue of where is the error: 

    $ csysdig -r crashloop.scap

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/06/image7.png" alt="csysdig" width="1564" height="717" class="alignleft size-full wp-image-4233" />][12] 

This time it was just a small typo error in the config file, but having total visibility of the system calls in a manageable interface where you can filter, search and visualize the dump data can help you debug <a href="https://sysdigrp2rs.wpengine.com/blog/container-isolation-gone-wrong/" target="_blank"> much harder problems</a>. 

### Nginx Dropped connections  {#nginx-dropped-connections}

If Nginx is dropping connections maybe it’s just malformed client requests, but if it’s happening frequently it may signal other issues like an anomalous requests overflow surpassing your <a href="https://www.nginx.com/blog/tuning-nginx/" target="_blank"> Nginx configured limits </a> (DDoS attacks, for example) or <a href="https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/" target="_blank">resource limits</a> on the pod. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/06/image8.png" alt="dropped conn" width="874" height="563" class="alignleft size-full wp-image-4234" />][13] 

<sub>Average of <code>nginx.net.conn_dropped_per_s</code> higher than 5 for the last 10 minutes.</sub>

### Nginx Service Quality  {#nginx-service-quality}

It is probably a good idea to fire an alert when your users are experiencing unresponsiveness, service response time metrics and HTTP service response time is available without code instrumentation for any microservice in your Kubernetes when using Sysdig Monitor: 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/06/Selection_136.png" alt="Backend lag" width="896" height="557" class="alignleft size-full wp-image-4235" />][14] 

<sub>Average of <code>net.http.request.time</code> higher than 0.5 seconds for the last 2 minutes.</sub>

### Nginx HTTP error codes  {#nginx-http-error-codes}

As we mentioned in the previous chapter, using Sysdig Monitor you can retrieve the HTTP response codes out of the box. The 4xx (client errors) and specially 5xx (server errors) are clear symptoms of backend problems. 

You probably want to raise an alarm if the amount of HTTP error codes raise above a certain limit specific to your scenario. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/06/image9.png" alt="Server Error" width="895" height="559" class="alignleft size-full wp-image-4236" />][15] 

<sub>Average of <code>net.http.request.count</code> with <code>net.http.statusCode</code> starting with '5' (5xx) higher than 5 for the last 2 minutes.</sub>

In the alert above, we will perform a separate check over each different Kubernetes service but you can also define an specific Pod or Kubernetes service in the *Scope* section. Using the scope conditions you can easily group several HTTP codes or just alert on specific ones. 

Imagine that instead of the 5xx catch-all described above, you want to alert specifically on *500 Internal Server Error* and *503 Service Unavailable* for a specific URL. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/06/image10.png" alt="billing api" width="890" height="589" class="alignleft size-full wp-image-4237" />][16] 

<sub>Average of <code>net.http.request.count</code> for <code>net.http.url</code> "company.com/billing" and <code>net.http.statusCode</code> 500 or 503 higher than 5 for the last 2 minutes.</sub>

## Conclusions  {#conclusions}

In this article we have covered the common failure or service degradation scenarios for the Nginx server in a Kubernetes deployment context. 

Being able to use any combination of: 

*   Lower level operative system metrics (memory or CPU usage, etc), 
*   HTTP parameters (return status code, URL, method, etc), 
*   Nginx metrics (connections per second, active connections, waiting connections, etc), 
*   Orchestrator-level information (Kubernetes and Docker events, CrashLoopBackOff status, metadata tags like namespaces, services deployments, replicaSets, etc), 

Provides a great deal of flexibility and meaningful context to fine-tune your alerting and notifications. Out of the box, no code instrumentation, no libraries, modified binaries or modified container images adding weight and complexity to your neat deployment. Haven't tried yet? Jump to our <a href="https://sysdigrp2rs.wpengine.com/" target="_blank"> 15-day free trial! </a>

 [1]: #alert-on-nginx-kubernetes-deployment
 [2]: #nginx-image-failing
 [3]: #nginx-dropped-connections
 [4]: #nginx-service-quality
 [5]: #nginx-http-error-codes
 [6]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/06/image1.png
 [7]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/06/Selection_135.png
 [8]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/06/image3.png
 [9]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/06/image4.png
 [10]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/06/image5.png
 [11]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/06/image6.png
 [12]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/06/image7.png
 [13]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/06/image8.png
 [14]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/06/Selection_136.png
 [15]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/06/image9.png
 [16]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/06/image10.png