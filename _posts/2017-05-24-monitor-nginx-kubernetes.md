---
ID: 4165
post_title: How to monitor Nginx on Kubernetes
author: Mateo Burillo
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/monitor-nginx-kubernetes/
published: true
post_date: 2017-05-24 12:22:51
---
In this article we are going to show how to monitor Nginx on Kubernetes, describing different use cases, peculiarities of running on this platform, relevant metrics and dashboards. We covered Nginx alerting in a second part: <a href="https://www.sysdigrp2rs.wpengine.com/blog/monitor-nginx-kubernetes-metrics-alerts/" target="_blank">Nginx metrics alerts</a>. 

## Why Nginx?  {#why-nginx}

<a href="https://www.nginx.com/" target="_blank" rel="noopener noreferrer"> Nginx </a> is a web server often deployed as a reverse proxy, load balancer and web cache. Designed for <a href="https://murphyswork.wordpress.com/2014/05/04/solve-c-10k-problem-with-nginx/" target="_blank" rel="noopener noreferrer"> high loads of concurrent connections </a> , it is famous for being fast, versatile, reliable and yet very light on resources. 

Nginx is a commonplace building block in containerized / cloud deployments. Actually, it is the top containerized application according to our last <a href="https://sysdigrp2rs.wpengine.com/blog/sysdig-docker-usage-report-2017/" target="_blank" rel="noopener noreferrer"> Docker usage report </a> with a sample size of 45,000 containers. 

You can use it as a classical web application server, as the gateway and balancer for a set of microservices or even as the Internet-facing entrypoint (like in a <a href="https://kubernetes.io/docs/concepts/services-networking/ingress/" target="_blank" rel="noopener noreferrer"> Ingress </a> controller on Kubernetes). When used as a load balancer, other common alternatives to Nginx are: HAProxy, the new and popular <a href="https://sysdigrp2rs.wpengine.com/blog/monitor-linkerd/" target="_blank" rel="noopener noreferrer"> Linkerd </a> , a public cloud service like AWS ELB or dedicated load-balancing devices.  
  
  
<span>[tweet_box design="default" float="none"]How to monitor #Nginx, especially in Kubernetes and Docker Environments. Key metrics explained. [/tweet_box]</span> 

## Nginx stub_status configuration on Kubernetes  {#nginx-stub_status-configuration-on-kubernetes}

In order to have Nginx expose its internal performance metrics and connection status metrics we need to enable the <a href="https://nginx.org/en/docs/http/ngx_http_stub_status_module.html" target="_blank" rel="noopener noreferrer"> stub_status module</a>. The commercial version, <a href="https://www.nginx.com/products/feature-matrix/" target="_blank" rel="noopener noreferrer">Nginx Plus</a>, provides some additional monitoring metrics, more fine grained connection status reporting or HTTP return code counters via the status module in addition to other features, but we will see later how Sysdig can give you some of that information as well. 

The Nginx <a href="https://hub.docker.com/_/nginx/" target="_blank" rel="noopener noreferrer"> official Docker image </a> and the binary packages for the most popular Linux distributions already include this module by default. 

To confirm that the module is available for the Nginx version you chose just run `
    nginx -V
   ` and look for the `
    --with-http_stub_status_module
   ` flag: 

    $ docker exec -ti nginx nginx -V
    nginx version: nginx/1.11.13
    built by gcc 4.9.2 (Debian 4.9.2-10) 
    built with OpenSSL 1.0.1t  3 May 2016
    TLS SNI support enabled
    configure arguments: --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx 
    ... 
    <b>--with-http_stub_status_module</b>
    ...
    

In order to apply the changes required to enable the module you can import the external configuration to the container using a Kubernetes <a href="https://kubernetes.io/docs/tasks/configure-pod-container/configmap/" target="_blank" rel="noopener noreferrer"> ConfigMap</a>. If you require several customizations you might consider creating your own custom Nginx image too. 

Here, we have taken the default `
    nginx.conf
   ` file, enabled the module `
    stub_status
   ` under `
    /nginx_status
   ` and we are also proxying the connections to a * wordpress * Kubernetes service, this is pretty much what we added: 

    server {
            server_name _;
    
            location /nginx_status {
              stub_status on;
              access_log  on;           
              allow all;  # REPLACE with your access policy
            }
    
            location / {
                proxy_pass http://wordpress:5000; # REPLACE with your service name and port
                proxy_set_header Host  $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_redirect off;
            }
    }
    

(see <a href="https://gist.github.com/mateobur/2535e6fc717301362e8805739fe1f496" target="_blank" rel="noopener noreferrer"> here the complete nginx.conf file </a> ) 

To expose this configuration, save it to a file and create the ConfigMap: 



    $ kubectl create configmap nginxconfig --from-file nginx.conf



You can then launch your Nginx containers in Kubernetes using a <a href="https://kubernetes.io/docs/concepts/workloads/controllers/deployment/" target="_blank" rel="noopener noreferrer"> Deployment </a> , <a href="https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/" target="_blank" rel="noopener noreferrer"> ReplicaSet </a> or <a href="https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/" target="_blank" rel="noopener noreferrer"> ReplicationController </a> and make them available through a <a href="https://kubernetes.io/docs/concepts/services-networking/service/" target="_blank" rel="noopener noreferrer"> Service </a> : 



    $ kubectl create -f nginxrc.yaml



(see <a href="https://gist.github.com/mateobur/88cfe9d8e33b6895de22fd3f4d5cbbe5" target="_blank" rel="noopener noreferrer"> here the complete nginxrc.yaml file </a> ) 

### Check Nginx status  {#check-nginx-status}

If you send a request to the configured URL, using `curl` for example, you should get an output like this: 

    $ curl nginx-wordpress/nginx_status
    Active connections: 6 
    server accepts handled requests
    100956 100956 101022 
    Reading: 0 Writing: 4 Waiting: 2
    

It’s a start, but most probably you want more advanced monitoring: historical data, graphs, dashboards, alerts... 

Now imagine that you could leverage Kubernetes metadata and labels when you configure all that monitoring. And then, imagine you could get application layer metrics like “Average request time per service” or some information typically found in processed logs like “top HTTP requests” or “slowest HTTP requests”. All together with the Nginx status module metrics. This is where Sysdig Monitor can help. 

You can start a <a href="https://sysdigrp2rs.wpengine.com/kubernetes" target="_blank" rel="noopener noreferrer"> Sysdig Monitor free trial </a> and this is how to install the agent using a Kubernetes <a href="https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/" target="_blank" rel="noopener noreferrer"> DaemonSet </a> as explained <a href="https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/206770633" target="_blank" rel="noopener noreferrer"> here</a>. Now let’s see what metrics we can find, how to design graphs using Kubernetes metadata and also how to create the corresponding alerts. 

## Monitoring Nginx metrics  {#monitoring-nginx-metrics}

We are going to review here which metrics are exposed by Nginx, what do they mean and some other related parameters required to understand and monitor Nginx behaviour and performance. 

### Nginx connections  {#nginx-connections}

Nginx provides metrics for the TCP connections that it receives from the clients. HTTP requests and responses go through these TCP connections. In HTTP/1 each request needs one connection but with HTTP/2 it is possible to make multiple requests in one single connection. In order to accelerate requests, these connections are sometimes kept open waiting for further requests from the same client, this is known as <a href="https://www.nginx.com/blog/http-keepalives-and-web-performance/" target="_blank" rel="noopener noreferrer"> Keepalive </a> . 

*   Network connectivity `nginx.can_connect`: Binary value checking the availability of the Nginx service. 
*   Current connections `nginx.net.connections`: Total number of active connections. 
*   New connections per second `nginx.net.conn_opened_per_s` : Rate of new connections opened per second. Comparing this value with the `nginx.net.connections` above you can gauge the effective throughput. 
*   Dropped connections per second `nginx.net.conn_dropped_per_s` : Rate of connections dropped by the Nginx server. Dropped connections may be caused by invalid client requests, rate limits or other Nginx configuration rules. 

<a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/Selection_002.png" target="_blank" rel="noopener noreferrer"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/Selection_002.png" alt="active nginx connections" width="1108" height="446" class="alignleft size-full wp-image-4196" /></a> 

<sub>Average of <code>nginx.net.conn_opened_per_s</code>, <code>nginx.net.connections</code> and <code>nginx.net.conn_dropped_per_s</code>.</sub>   
  
### Nginx request status  {#nginx-request-status}

Nginx provides a metric that shows the number of HTTP requests performed, therefore we can also get the number of requests per second. Each request goes through different states that you can get displayed as a rate. 

*   Requests per second `nginx.net.requests_per_s` : Rate of processed requests. 
*   Requests in reading state `nginx.net.reading` : Nginx is reading the client request (headers). 
*   Requests in waiting state `nginx.net.waiting` : Nginx is waiting/idle, may be waiting for the backends to process the response or for the client to close the connection. 
*   Requests in writing state `nginx.net.writing` : Nginx is writing the response back to the client. 

<a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/Selection_001.png" target="_blank" rel="noopener noreferrer"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/Selection_001.png" alt="nginx writing" width="1102" height="438" class="alignleft size-full wp-image-4198" /></a> 

<sub>Average of <code>nginx.net.reading</code>, <code>nginx.net.waiting</code> and <code>nginx.net.writing</code>.</sub>   
  
### Nginx HTTP application metrics  {#nginx-http-application-metrics}

Nginx+ provides some visibility on HTTP response codes but if we want to further monitor Nginx  metrics like request time, we need to enable <a href="https://nginx.org/en/docs/http/ngx_http_log_module.html#log_format" target="_blank" rel="noopener noreferrer"> $request_time </a> in the log module and then calculate that metric through a logging system. 

Sysdig Monitor makes this way easier. There is no need to configure a complex setup to calculate metrics from logs. It is done automatically, just by decoding the HTTP protocol extracted from the payloads of `
    read()` and `write()` system calls of file descriptor sockets opened by Nginx. This way, Sysdig can provide you some interesting HTTP protocol application layer information without any kind of code instrumentation: 

*   Top URLs `net.request.count|net.http.request.count` segmented by `net.http.url`: Rate of hits per HTTP URL. Useful to monitor user behaviour, popular resources and to detect anomalous connections. 
*   Slowest URLs `net.http.url` segmented by `net.http.request.time`: URLs that take the most time to complete, on average. These are possible bottlenecks that you need to optimize to gain overall responsiveness. 
*   HTTP response codes `net.http.statusCode`: HTTP response codes provide a lot of meaningful information about your backends health, will be further detailed in [this section][1]. 
*   Service request time, both average (`net.request.time|net.http.request.time` segmented by `kubernetes.service.name`). 

<a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/Selection_102.png" target="_blank" rel="noopener noreferrer"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/Selection_102.png" alt="nginx response codes" width="1102" height="444" class="alignleft size-full wp-image-4199" /></a> 

<sub>Rate of <code>net.request.count</code> segmented by <code>net.http.statusCode</code>.</sub>   
  
This table summarizes the metrics and where they come from: 

<style type="text/css">
  .tg  {border-collapse:collapse;border-spacing:0;}
.tg td{font-family:Arial, sans-serif;font-size:14px;padding:10px 5px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;}
.tg th{font-family:Arial, sans-serif;font-size:14px;font-weight:normal;padding:10px 5px;border-style:solid;border-width:1px;overflow:hidden;word-break:normal;}
.tg .tg-uqo3{background-color:#efefef;text-align:center;vertical-align:top}
.tg .tg-baqh{text-align:center;vertical-align:top}
.tg .tg-yzt1{background-color:#efefef;vertical-align:top}
.tg .tg-yw4l{vertical-align:top}
</style>

<table class="tg">
  <tbody>
    <tr>
      <th class="tg-yzt1">
      </th>
      
      <th class="tg-uqo3" colspan="6">
        <b>Source</b>
      </th>
    </tr>
    
    <tr>
      <td class="tg-yzt1">
        <b>Metric</b>
      </td>
      
      <td class="tg-yzt1">
        <b>Nginx stub_status module</b>
      </td>
      
      <td class="tg-yzt1">
        <b>Nginx+ status module</b>
      </td>
      
      <td class="tg-yzt1" colspan="4">
        <b>Logs / APM / Sysdig visibility</b>
      </td>
    </tr>
    
    <tr>
      <td class="tg-yw4l">
        accepts
      </td>
      
      <td class="tg-baqh">
        ✔
      </td>
      
      <td class="tg-baqh">
        ✔<br />(accepted)
      </td>
      
      <td class="tg-baqh" colspan="4">
        ✔
      </td>
    </tr>
    
    <tr>
      <td class="tg-yw4l">
        handled
      </td>
      
      <td class="tg-baqh">
        ✔
      </td>
      
      <td class="tg-baqh">
        ✔
      </td>
      
      <td class="tg-baqh" colspan="4">
        ✔
      </td>
    </tr>
    
    <tr>
      <td class="tg-yw4l">
        dropped
      </td>
      
      <td class="tg-baqh">
        ✔<br />(calculated)
      </td>
      
      <td class="tg-baqh">
        ✔
      </td>
      
      <td class="tg-baqh" colspan="4">
        ✔
      </td>
    </tr>
    
    <tr>
      <td class="tg-yw4l">
        active
      </td>
      
      <td class="tg-baqh">
        ✔<br />(includes waiting)
      </td>
      
      <td class="tg-baqh">
        ✔<br />(excludes waiting)
      </td>
      
      <td class="tg-baqh" colspan="4">
        ✔
      </td>
    </tr>
    
    <tr>
      <td class="tg-yw4l">
        waiting
      </td>
      
      <td class="tg-baqh">
        ✔
      </td>
      
      <td class="tg-baqh">
        ✔<br />(idle)
      </td>
      
      <td class="tg-baqh" colspan="4">
        ✔
      </td>
    </tr>
    
    <tr>
      <td class="tg-yw4l">
        reading
      </td>
      
      <td class="tg-baqh">
        ✔
      </td>
      
      <td class="tg-baqh">
        ✔
      </td>
      
      <td class="tg-baqh" colspan="4">
        ✔
      </td>
    </tr>
    
    <tr>
      <td class="tg-yw4l">
        writing
      </td>
      
      <td class="tg-baqh">
        ✔
      </td>
      
      <td class="tg-baqh">
        ✔
      </td>
      
      <td class="tg-baqh" colspan="4">
        ✔
      </td>
    </tr>
    
    <tr>
      <td class="tg-yw4l">
        requests
      </td>
      
      <td class="tg-baqh">
        ✔
      </td>
      
      <td class="tg-baqh">
        ✔<br />(total)
      </td>
      
      <td class="tg-baqh" colspan="4">
        ✔
      </td>
    </tr>
    
    <tr>
      <td class="tg-yw4l">
        4xx codes
      </td>
      
      <td class="tg-baqh">
      </td>
      
      <td class="tg-baqh">
        ✔
      </td>
      
      <td class="tg-baqh" colspan="4">
        ✔
      </td>
    </tr>
    
    <tr>
      <td class="tg-yw4l">
        5xx codes
      </td>
      
      <td class="tg-baqh">
      </td>
      
      <td class="tg-baqh">
        ✔
      </td>
      
      <td class="tg-baqh" colspan="4">
        ✔
      </td>
    </tr>
    
    <tr>
      <td class="tg-yw4l">
        request time
      </td>
      
      <td class="tg-baqh">
      </td>
      
      <td class="tg-baqh">
      </td>
      
      <td class="tg-baqh" colspan="4">
        ✔
      </td>
    </tr>
  </tbody>
</table>

### System and resource metrics for Nginx  {#system-and-resource-metrics-for-nginx}

In addition, we shouldn’t forget about monitoring the system resources that Nginx needs to perform properly: 

*   Percentage of used CPU `
     cpu.used.percent`. 
*   Load average, `load.average.percpu.1m, 5m, 15m` matching the usual load measurement periods. 
*   Memory used both in absolute terms `memory.bytes.used` (although it is named ‘bytes’ the graph will adjust to the best human readable scale like Mega or Giga) or percentage `memory.used.percent`. 
*   IOPS total `file.iops.total`. For example, if you are using a local content cache. 
*   Network bytes activity `net.bytes.total`, `net.bytes.in`, `net.bytes.out`, useful to know when are you going to reach net limits and need to scale up. 
*   Net error count `
     net.error.count
    ` connectivity problems, not to be confused with the [HTTP-level error codes][1]. 

<a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/Selection_104.png" target="_blank" rel="noopener noreferrer"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/Selection_104.png" alt="nginx bytes" width="1100" height="434" class="alignleft size-full wp-image-4200" /></a> 

<sub>Rate of <code>net.bytes.total</code> segmented by <code>kubernetes.pod.name</code>.</sub>   
  
OK, so now that we have discussed the most important metrics, let’s see how you can visualize and work with them.  [tweet_box design="default" float="none"]How to monitor #Nginx, bringing service and application metrics without code instrumentation[/tweet_box] 

## Nginx Dashboards  {#nginx-dashboards}

You can create an Nginx dashboard on the *Dashboards* tab, *ADD DASHBOARD* and scroll down or search for ‘nginx’ template. Using *Scope* you can limit the visualization to any Kubernetes entity like a node, a namespace, a Service or a Deployment, even something like an AWS region, that’s possible too! 

<a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/image1.png" target="_blank" rel="noopener noreferrer"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/image1.png" alt="nginx dashboard" width="2205" height="1039" class="alignleft size-full wp-image-4179" /></a>  Once the dashboard has been created you can customize it: add or remove graphs, change the scope or segmentation of each graph, see events on the graphs, resize them, etc. 

The default dashboard includes: connection status (writing, reading, waiting), CPU load, network traffic, requests per second, top URLs, slowest URLs, active connections, dropped connections and response codes. 

The HTTP dashboard template includes some more application layer metrics like: average request time, maximum request time, or requests type (GET, POST, etc). You could merge the metrics that are more interesting for you into a single panel using the *Copy Panel* icon <a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/pin.png" target="_blank" rel="noopener noreferrer"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/pin.png" alt="pin" width="26" height="33" class="alignleft size-full wp-image-4168" /></a> and grouping them into a different dashboard. 

<a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/image2.png" target="_blank" rel="noopener noreferrer"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/image2.png" alt="http dash" width="2244" height="953" class="alignleft size-full wp-image-4178" /></a> 

In a more dynamic fashion, similar views to these dashboards are available in the *Explore* tab, just select the scope that you want to visualize and apply the * Nginx * view, * HTTP * view or * HTTP Top Requests * view. 

## Using Kubernetes labels in graphs to monitor Nginx  {#using-kubernetes-labels-in-graphs-to-monitor-nginx}

When using microservices in orchestration platforms like Kubernetes you need to monitor Nginx metrics both at the service level but also individually per container or per pod. Kubernetes metadata such as labels are very helpful for this, but using Sysdig you can use any of the available metadata to do this kind of segmentation, either from Kubernetes, Docker or any cloud provider like AWS (think of showing a metric segmented by availability zone). 

### Segmentation by Kubernetes Namespace  {#segmentation-by-kubernetes-namespace}

For example, let’s assume that you have three different Kubernetes namespaces to completely separate the * development * , * staging * and * production * environments. Now you want to know how many requests per second each environment is receiving, so we can segment by `kubernetes.namespace.name`: 

<a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/Selection_106.png" target="_blank" rel="noopener noreferrer"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/Selection_106.png" alt="request per second" width="1083" height="427" class="alignleft size-full wp-image-4177" /></a> 

<sub>Average of <code>nginx.net.request_per_s</code> segmented by <code>kubernetes.namespace.name</code>.</sub>   
  
A similar example could be that, as part of your CI/CD pipeline, you need to have a real time benchmark comparing your current production code and ‘N+1 staging’ code. 

Let’s compare again processed requests per second, replaying real traffic to the staging environment: 

<a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/image4.png" target="_blank" rel="noopener noreferrer"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/image4.png" alt="staging vs prod" width="1087" height="437" class="alignleft size-full wp-image-4176" /></a> 

<sub>Average of <code>nginx.net.request_per_s</code> segmented by <code>kubernetes.namespace.name</code>.</sub>   
  
Doing the same kind of segmentation on the metric that shows HTTP error responses would also be interesting for this scenario: 

<a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/image5.png" target="_blank" rel="noopener noreferrer"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/image5.png" alt="staging vs prod" width="1076" height="430" class="alignleft size-full wp-image-4175" /></a> 

<sub>Rate of <code>net.http.error.count</code> segmented by <code>kubernetes.namespace.name</code>.</sub>   
  
Oops! looks like there are some unresolved issues in our staging code, we cannot move it to production. 

### Segmentation by Kubernetes Service  {#segmentation-by-kubernetes-service}

One of the most wished-for features when using microservices is being able to monitor each of the services that build up the entire user-facing application. Usually we want to find which is the slowest microservice: who is the bottleneck? 

In our case, we use a separate ReplicationController per service, this way we can easily segment the HTTP requests. Typically you can use here `kubernetes.replicationController.name` or `kubernetes.service.name` tags to segment any metric in the time series graph. 

You want to know which of your services needs more bandwidth: 

<a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/image6.png" target="_blank" rel="noopener noreferrer"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/image6.png" alt="bandwidht" width="1086" height="431" class="alignleft size-full wp-image-4174" /></a> 

<sub>Rate of <code>net.bytes.total</code> segmented by <code>kubernetes.replicationController.name</code>.</sub>   
  
Or receives most requests per second: 

<a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/image7.png" target="_blank" rel="noopener noreferrer"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/image7.png" alt="requests per second" width="1098" height="444" class="alignleft size-full wp-image-4173" /></a> 

<sub>Rate of <code>nginx.net.request_per_s</code> segmented by <code>kubernetes.replicationController.name</code>.</sub>   
  
Another common use case is that you want to know which service takes more time to process user requests. Again, Sysdig transparent metric collection allows you to see response times for each microservice without any kind of code instrumentation or 'sidecar monitoring container' adding complexity to your pods. This is probably the most useful troubleshooting first step to find out what's going on within a microservices application. 

<a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/Selection_105.png" target="_blank" rel="noopener noreferrer"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/Selection_105.png" alt="average time" width="1087" height="438" class="alignleft size-full wp-image-4172" /></a> 

<sub>Rate of <code>net.http.request.time</code> segmented by <code>kubernetes.replicationController.name</code>.</sub>   
  
Or, you noticed there were some issues using your services at one specific point in time and want to check HTTP errors per service: 

<a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/image9.png" target="_blank" rel="noopener noreferrer"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/image9.png" alt="http errors" width="1097" height="444" class="alignleft size-full wp-image-4171" /></a> 

<sub>Rate of <code>net.http.error.count</code> segmented by <code>kubernetes.replicationController.name</code>.</sub>   
  
### Segmentation by Kubernetes Pod  {#segmentation-by-kubernetes-pod}

Want a separate graph line per container? just segment by `pod.name`. 

<a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/Peek-2017-05-20-13-39.gif" target="_blank" rel="noopener noreferrer"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/Peek-2017-05-20-13-39.gif" alt="pod name segmentation" width="2470" height="1010" class="alignleft size-full wp-image-4180" /></a> 

<sub>Segment by <code>host.hostName</code> => Segment by <code>pod.name</code>.</sub>   
  
This is especially interesting when you are looking exactly which container is not behaving as it should compared with the rest of containers in your service. You can also easily create a board to assess the individual performance of your Nginx pods: 

<a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/image10.png" target="_blank" rel="noopener noreferrer"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/image10.png" alt="pod dash" width="2203" height="1038" class="alignleft size-full wp-image-4170" /></a> 

### Segmentation by HTTP response code and HTTP method  {#segmentation-by-http-response-code-and-http-method}

We have mentioned before how interesting is being able to use Nginx metrics, coupled with application metrics like response time, HTTP URL, response code or HTTP method. 

Monitoring the <a href="https://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html" target="_blank" rel="noopener noreferrer"> HTTP methods </a> (POST, GET, PUT, PATCH, DELETE…) can be really useful to audit how the clients are using your REST APIs. 

The <a href="https://en.wikipedia.org/wiki/List_of_HTTP_status_codes" target="_blank" rel="noopener noreferrer"> HTTP response codes </a> reveal a lot of information on what’s going on with your application or API. You should look not only for the famous * 404 Not Found * or * 500 Internal Server Error * , there are also other meaningful errors like disallowed methods, bad gateway or gateway timeouts: 

<a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/image11.png" target="_blank" rel="noopener noreferrer"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/image11.png" alt="http errors" width="1080" height="429" class="alignleft size-full wp-image-4169" /></a> 

<sub>Rate of <code>net.request.count</code> segmented by <code>net.http.statusCode</code>.</sub>   
  
As you might have guessed already, it is probably a good idea to set an alert if the monitor is detecting too many 4xx 5xx error codes. 

## Nginx Plus

<a href="https://www.nginx.com/products/nginx/" target="_blank">Nginx Plus</a> (or Nginx+) is the commercial version of the Nginx server. It builds several enterprise features on top of the opensource version like health-checks, advanced load balancing, clustering, multimedia extensions and, specifically related with the context of this article, **additional metrics**.

This version also features its own management dashboard:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/Selection_263.png" alt="Nginx Plus Dashboard" width="1821" height="728" class="alignleft size-full wp-image-4597" />][2] 


Of course, you can integrate Nginx Plus with Sysdig Monitor and take advantage of all these extra metrics combining them with container visibility and orchestration context. For example, you can fire an alarm when the `nginx.upstream.peers.health_checks.unhealthy` goes above certain threshold for the *production* `kubernetes.namespace.name`

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/Selection_264.png" alt="Nginx Plus alert" width="988" height="605" class="alignleft size-full wp-image-4598" />][3] 


For Nginx Plus to work with Sysdig, you first need to expose the status endpoint. For this example we are going to add a separate metrics virtualhost:

<pre class="editor-colors lang-text"><div class="line">
  <span class="syntax--text syntax--plain"><span class="syntax--meta syntax--paragraph syntax--text"><span>server&nbsp;{</span></span></span>
  
</div>

<div class="line">
  <span class="syntax--text syntax--plain"><span>&nbsp;&nbsp;</span><span class="syntax--meta syntax--paragraph syntax--text"><span>listen&nbsp;0.0.0.0:8080;</span></span></span>
  
</div>

<div class="line">
  <span class="syntax--text syntax--plain"><span class="syntax--meta syntax--paragraph syntax--text"><span>&nbsp;&nbsp;root&nbsp;&nbsp;&nbsp;/usr/share/nginx/html;</span></span></span>
  
</div>

<div class="line">
  <span class="syntax--text syntax--plain"><span>&nbsp;</span></span>
  
</div>

<div class="line">
  <span class="syntax--text syntax--plain"><span>&nbsp;&nbsp;</span><span class="syntax--meta syntax--paragraph syntax--text"><span>location&nbsp;/status&nbsp;{</span></span></span>
  
</div>

<div class="line">
  <span class="syntax--text syntax--plain"><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span class="syntax--meta syntax--paragraph syntax--text"><span>status;</span></span></span>
  
</div>

<div class="line">
  <span class="syntax--text syntax--plain"><span class="syntax--meta syntax--paragraph syntax--text"><span>&nbsp;&nbsp;&nbsp;&nbsp;allow&nbsp;all;&nbsp;#&nbsp;replace&nbsp;with&nbsp;your&nbsp;custom&nbsp;access&nbsp;policy&nbsp;&nbsp;&nbsp;&nbsp;</span></span></span>
  
</div>

<div class="line">
  <span class="syntax--text syntax--plain"><span>&nbsp;&nbsp;</span><span class="syntax--meta syntax--paragraph syntax--text"><span>}</span></span></span>
  
</div>

<div class="line">
  <span class="syntax--text syntax--plain"><span>&nbsp;</span></span>
  
</div>

<div class="line">
  <span class="syntax--text syntax--plain"><span>&nbsp;&nbsp;</span><span class="syntax--meta syntax--paragraph syntax--text"><span>location&nbsp;=&nbsp;/status.html&nbsp;{</span></span></span>
  
</div>

<div class="line">
  <span class="syntax--text syntax--plain"><span class="syntax--meta syntax--paragraph syntax--text"><span>&nbsp;&nbsp;}</span></span></span>
  
</div>

<div class="line">
  <span class="syntax--text syntax--plain"><span class="syntax--meta syntax--paragraph syntax--text"><span>}</span></span></span>
  
</div></pre>

Port *8080* url `<yourdomain>/status` will publish the raw JSON metrics (you can actually display them in your browser but it’s not suited for human consumption). If you access `<yourdomain>/status.html` you will be able to see the dashboard we mentioned above.

Once you have enabled the status endpoint, you need to make the Sysdig agent aware of it. Edit the `/opt/draios/etc/dragent.yaml` additional configuration file. For this example we are going to add the following section:

<pre class="editor-colors lang-text"><div class="line">
  <span class="syntax--text syntax--plain"><span class="syntax--meta syntax--paragraph syntax--text"><span>customerid:&nbsp;&lt;your_customer_id&gt;</span></span></span>
  
</div>

<div class="line">
  <span class="syntax--text syntax--plain"><span class="syntax--meta syntax--paragraph syntax--text"><span>tags:&nbsp;&lt;yourtags&gt;</span></span></span>
  
</div>

<div class="line">
  <span class="syntax--text syntax--plain"><span class="syntax--meta syntax--paragraph syntax--text"><span>app_checks:</span></span></span>
  
</div>

<div class="line">
  <span class="syntax--text syntax--plain"><span>&nbsp;&nbsp;</span><span class="syntax--meta syntax--paragraph syntax--text"><span>-&nbsp;name:&nbsp;nginx</span></span></span>
  
</div>

<div class="line">
  <span class="syntax--text syntax--plain"><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span class="syntax--meta syntax--paragraph syntax--text"><span>check_module:&nbsp;nginx</span></span></span>
  
</div>

<div class="line">
  <span class="syntax--text syntax--plain"><span class="syntax--meta syntax--paragraph syntax--text"><span>&nbsp;&nbsp;&nbsp;&nbsp;pattern:</span></span></span>
  
</div>

<div class="line">
  <span class="syntax--text syntax--plain"><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span class="syntax--meta syntax--paragraph syntax--text"><span>exe:&nbsp;"nginx:&nbsp;worker&nbsp;process"&nbsp;#&nbsp;nginx&nbsp;overwrites&nbsp;argv[0]&nbsp;that&nbsp;is&nbsp;parsed&nbsp;as&nbsp;exe&nbsp;on&nbsp;sysdig</span></span></span>
  
</div>

<div class="line">
  <span class="syntax--text syntax--plain"><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span class="syntax--meta syntax--paragraph syntax--text"><span>conf:</span></span></span>
  
</div>

<div class="line">
  <span class="syntax--text syntax--plain"><span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span class="syntax--meta syntax--paragraph syntax--text"><span>nginx_status_url:&nbsp;"http://&lt;yourdomain&gt;:8080/status/"</span></span></span>
  
</div>

<div class="line">
  <span class="syntax--text syntax--plain"><span>&nbsp;&nbsp;&nbsp;&nbsp;</span><span class="syntax--meta syntax--paragraph syntax--text"><span>log_errors:&nbsp;false</span></span></span>
  
</div></pre>

This configuration YAML file will merge with the default agent configuration. You have more information on customizing your Sysdig agent <a href="https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/206443225-How-can-I-edit-the-agent-s-configuration-file-" target="_blank">here</a>.

Restart the agent:

`sudo service dragent restart`

And voila! you should start receiving the Nginx Plus-specific metrics in your Sysdig Monitor interface right away.

Some interesting metrics:

*   Server Zone metrics (i.e. `nginx.server_zone.responses.total`), connections between the end client and the Nginx front servers.
*   Upstream metrics (i.e. `nginx.upstream.peers.active`), to monitor connection between the Nginx front servers and web backends. 
    *   Upstream health checks (i.e. `nginx.upstream.peers.health_checks.fails`)
*   SSL handshake information (i.e. `nginx.ssl.session_reuses`).
*   Extended connections metrics like dropped connections `nginx.connections.dropped`.
*   Total number of abnormally terminated and respawned Nginx child processes `nginx.processes.respawned`.

This is an example Nginx Plus dashboard using some of those metrics:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/Workspace-1_261.png" alt="Nginx Plus Sysdig Monitor Dashboard" width="2225" height="1154" class="alignleft size-full wp-image-4599" />][4] 


## Conclusions  {#conclusions}

Nginx is a powerhouse of the cloud. Its flexibility and conciseness means that is easy to do simple deployments and affordable to do more complex ones. It is also so light on resources you probably don’t have to think twice about deploying a lot of replicas to ensure high availability. 

Nginx servers are usually in a privileged position inside your infrastructure to analyze service responsiveness, detect bottlenecks and predict backend failures, don’t miss the opportunity to make the best of this information. Sysdig can monitor Nginx metrics with application layer metrics like service response time, HTTP methods, response code and top or slowest URls, providing a complete visibility of your Nginx and related microservices without doing any kind of code instrumentation, just looking at the system calls! 

Enjoyed this? Move into the second part of How to monitor Nginx on Kubernetes: <a href="https://www.sysdigrp2rs.wpengine.com/blog/monitor-nginx-kubernetes-metrics-alerts/" target="_blank">Metrics alerts</a>, where you will learn Nginx alert configuration matching the common failure points and the use cases we described on this article.

 [1]: #segmentation-by-http-response-code-and-http-method
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/Selection_263.png
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/Selection_264.png
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/Workspace-1_261.png