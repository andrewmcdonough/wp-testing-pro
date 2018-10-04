---
ID: 4105
post_title: >
  How to monitor Linkerd the microservices
  proxy
author: Mateo Burillo
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/monitor-linkerd/
published: true
post_date: 2017-05-10 09:55:29
---
In this article we are going to deploy and monitor Linkerd as the default proxy for microservices communication inside a Kubernetes cluster. Common Linkerd metrics, failure points and their related alerts will be presented. 

Containerization and the closely related concept of microservices are turning the IT world upside down, and for a good set of reasons. If you are reading this, you are probably already familiar with the benefits and exciting ideas of [ the new stack][1]. Of course, new architecture design principles come with its own share of challenges like microservice inter-communication or traceability. 

### Microservices communication  {#microservices-communication}

Containers are numerous, mostly anonymous and ephemeral. This is a perfect fit for scalability, redundancy and high availability but makes connection between the different entities somewhat more complex. Kubernetes services allow to [ expose a Service][2], abstracting the network details while providing basic load balancing and high availability. This is implemented as a simple connection round-robin using iptables underneath. Often, more latency or application aware methods are required. Should the client code be in charge of timeouts and/or retries?. Routing external requests coming from the Internet to the cluster services ([Ingress][3]) is not an obvious process either. 

### Service traceability  {#service-traceability}

Containers and microservices are really interesting concepts for IT engineers, not so much for the final user. Users will evaluate how responsive and reliable the service is as a whole. Top-line metrics of a single microservice are a must, being able to monitor the complete round trip of a client request is often required. 

## What is Linkerd?  {#what-is-linkerd}

[ Linkerd ][4] is an open source, scalable service mesh proxy. It aims to solve some of the challenges we just discussed: the emerging communication complexities and traceability on large scale deployments. 

Main features of Linkerd are: 

*   Advanced load-aware balancer with different forwarding strategies. 
*   Abstracts away the service discovery mechanism. 
*   Dynamic request routing. By tweaking the *dtab* router you can dynamically shift traffic during A/B testing, staging/production, etc. 
*   Automatic TLS tunneling for HTTP requests. 
*   Manages request retries, timeouts and deadlines, freeing other services or clients from implementing this network logic. 
*   Circuit breaking, removing unhealthy service instances from the pool. 

## Linkerd deployment options  {#linkerd-deployment-options}

A straightforward option is to deploy Linkerd in *convoy* mode, this is, as an extra container attached to every pod, but depending on your scale it can become expensive and you will also need to modify your current pod definitions. 

Another possibility is to deploy it using a [ DaemonSet][5], this will deploy exactly one Linkerd pod per Kubernetes node. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/linkerd1.png" alt="linkerd DaemonSet" width="793" height="580" class="alignleft size-full wp-image-4107" />][6]   
<sub>Image source: https://blog.buoyant.io/2016/10/14/a-service-mesh-for-kubernetes-part-ii-pods-are-great-until-theyre-not/</sub> 

Each time a pod wants to communicate it will forward traffic to its node-local Linkerd instance (Kubernetes [downward API][7] provides this information), and Linkerd will take care of the routing from that point on. 

This will be the deployment mode used to illustrate the how to monitor Linkerd in this article, Linkerd uses three ports by default: 

*   Incoming traffic 4141 
*   Outgoing traffic 4140 
*   Local admin web 9990 (see monitoring section below) 

To deploy Linkerd on Kubernetes we used a service like this: 

<script src="https://gist.github.com/1d99b7bf411f78cd3dbe696ff3c555e2.js">
  </script> <noscript>
  <pre><code>
File: l5d-service.yml
---------------------

---
apiVersion: v1
kind: Service
metadata:
  name: l5d
spec:
  selector:
    app: l5d
  type: LoadBalancer
  ports:
  - name: outgoing
    port: 4140
  - name: incoming
    port: 4141
  - name: admin
    port: 9990

</code></pre>
</noscript>

[ And here ][8] is the Linkerd complete DaemonSet and ConfigMap config file (based on buoyant.io [ series of Linkerd articles][9]). 

## Monitor Linkerd  {#monitor-linkerd}

### Integrated monitoring interfaces  {#integrated-monitoring-interfaces}

As mentioned above, a native web admin interface comes bundled out of the box: 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/linkerd2.png" alt="linkerd webadmin" width="1554" height="951" class="alignleft size-full wp-image-4116" />][10] 

Using this web interface, you can get an overview of traffic and current requests, retries, failed requests, etc. The *dtab* routing section is also quite interesting, you can trace the path of a certain request and dynamically change the routing rules. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/linkerd3.png" alt="linkerd dtab" width="1302" height="855" class="alignleft size-full wp-image-4115" />][11] 

### Sysdig Monitor  {#sysdig-monitor}

In order to monitor Linkerd with Sysdig Monitor we need to deploy the Sysdig Monitor agent in your Kubernetes cluster, like Linkerd, as a DaemonSet. If you haven’t done this before, just check out the instructions from [ here ][12] , it’s just one `kubectl` command. 

#### statsd teleport!  {#statsd-teleport}

Linkerd can easily leverage statsd to expose the context-specific metrics. You can either send the metrics to a *sidecar* container (a exporter), have a dedicated statsd collector pod and instruct all the other services to send the metrics there, or even much easier, just forward all your metrics to a non-existent localhost port as shown in the Linkerd ConfigMap used above: 

<script src="https://gist.github.com/a27e2921caa0f32c24717a5b51563b95.js">
  </script> <noscript>
  <pre><code>
File: statsd.yml
----------------

 telemetry:
    - kind: io.l5d.statsd
      experimental: true
      prefix: linkerd
      hostname: 127.0.0.1

      port: 8125
      gaugeIntervalMs: 10000
      sampleRate: 0.01
</code></pre>
</noscript>

** Wait, what?! ** 

The Sysdig Monitor agent features [ passive statsd collection ][13] (or statsd teleport), the UDP packages will be eventually discarded, but Sysdig kernel module has already captured and forwarded the relevant information to the monitoring agent container, where it gets enriched with a set of tags and then sent to the Sysdig Monitor backend. 

Accessing the Sysdig Monitor web interface and filtering by the ‘linkerd’ string, you will find a wealth of metrics available: 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/linkerd4.png" alt="linkerd metrics" width="2500" height="502" class="alignleft size-full wp-image-4114" />][14] 

Very conveniently, Linkerd also yields a different set of metrics per namespace and service, such as `
    l5d_k8s.<strong>production</strong>.http.<strong>nginx</strong>.retries.budget`, where `production` is one of our namespaces and `nginx` one of our services. 

You can easily create your own Linkerd dashboard with the parameters that you consider most relevant for your case: 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/linkerd5.png" alt="example dash" width="2206" height="1039" class="alignleft size-full wp-image-4113" />][15] 

### Most important linkerd metrics  {#most-important-linkerd-metrics}

*   Request latency (`svc.'yourservice'.request_latency_ms`): aggregating all the successive calls, see section below. 
*   Request latency percentiles (`svc.'yourservice'.request_latency_ms.p50, p95, p99`): useful to get a grip of standard deviation and worst case scenario. 
*   Service failures (`svc.'yourservice'.failures`). 
*   Total active connections (`'yourservice'.connections`). 
*   Requests pending (`svc.'yourservice'.pending`): queue size and concurrency. 
*   Total requests (`svc.'yourservice'.requests`). 
*   Service unhealthy (`failfast.unhealthy_for_ms, failfast.unhealthy_num_tries`): [ Circuit breaking ][16]capabilities. 
*   Loadbalancer removed node (`loadbalancer.removes`): one of the nodes providing a service removed from the loadbalancer. 
*   Loadbalancer available nodes (`'yourservice'.loadbalancer.available`): number of nodes that the balancer considers alive for a given service. 
*   Service bandwidth (`'yourservice'.received_bytes, 'yourservice'.sent_bytes`). 
*   Connection retries (`svc.'yourservice'.retries.total, .retries.budget`). 

### Monitoring microservices aggregating latency  {#monitoring-microservices-aggregating-latency}

Imagine the following common scenario: you have an edge proxy A which, depending on the request URL will open a connection to service B or service C. B will open a successive connection with D to complete the request. C will open a connection with E and E with F. 

A ⇒ Linkerd ⇒ B ⇒ Linkerd ⇒ D 

or 

A ⇒ Linkerd ⇒ C ⇒ Linkerd ⇒ E ⇒ Linkerd ⇒ F 

Each request will finish when the successive requests returns, which allows you to measure the latency of individual microservices (D or F), entire services (B+D; C+E+F) or global average response time (A). 

Consider the example dashboard below: 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/linkerd6.png" alt="services latency" width="2192" height="1034" class="alignleft size-full wp-image-4112" />][17] 

You can monitor: 

*   The aggregated edge proxy latency (Internet-facing HAproxy, for example) -top left panel. 
*   Complete service latency (internal Apache reverse proxy + ownCloud + several database backends) - top right panel. 
*   Log backend individual microservice latency - bottom right panel. 

### Distributed tracing with Zipkin  {#distributed-tracing-with-zipkin}

Linkerd also supports forwarding connection data to distributed tracing systems like [ Zipkin][18], being able to monitor timing and latency for each request across all the different microservices. 

## Linkerd alerts  {#linkerd-alerts}

Now that you have imported all the metrics and Linkerd context-specific data and tags, next step is to configure the relevant alerts: 

*   [Service latency][19] 
*   [Connection failure][20] 
*   [Available endpoints][21] 
*   [Retries budget][22] 

### Service latency  {#service-latency}

An early indicator of high loads or deficiencies in any of the backends, it is a good practice to set an upper bound latency for your business-critical services. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/linkerd7.png" alt="latency alert" width="991" height="654" class="alignleft size-full wp-image-4111" />][23] 

(Metric name (without prefix): `
    production.http.frontend.request_latency_ms`) 

### Connection failure  {#connection-failure}

If Linkerd is failing to connect with a specific service, it is probably a clear indicator of issues. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/linkerd8.png" alt="connection failure" width="994" height="661" class="alignleft size-full wp-image-4110" />][24] 

Depending on the average load and responsivity of your system, your acceptable rate of connection failures will vary. 

### Available endpoints  {#available-endpoints}

The Linkerd load balancer keeps a table of alive endpoints per service, you may want to fire an alert if this value drops. For example, if you have declared in your ReplicaSet that some service is composed of 10 instances, send an alert if 7 or less are currently available: 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/linkerd9.png" alt="available endpoints" width="992" height="649" class="alignleft size-full wp-image-4109" />][25] 

This alert bears some similarities with the running pods versus desired pods alert we described in our Monitoring Kubernetes guide: [ best practices for alerting on Kubernetes ][26] . 

### Retries budget  {#retries-budget}

Linkerd keeps a global ‘Retry Budget’ which is spent with every connection retry and regenerates automatically over time, you can keep an eye on this metric and be warned if the number of connection retries is anomalous. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/linkerd10.png" alt="retry budget" width="996" height="659" class="alignleft size-full wp-image-4108" />][27] 

(Metric name (without prefix): `
    k8s.production.http.frontend.retries.budget`)

## Conclusions  {#conclusions}

Linkerd is an infrastructure add-on that can certainly add some extra capabilities over the default Kubernetes service balancers, handling the communication between your microservices and becoming a critical part of your infrastructure. 

An interesting benefit of this approach is being able to free individual containers from networking concerns like retries, timeouts or cypher. This enables them to be lighter, simpler and more deployment-agnostic, very much in line with the microservices approach. 

Monitoring Linkerd using Sysdig Monitor is really easy thanks to Sysdig statsd teleport: you can automatically import all the metrics with the required context, gaining visibility on the communication between the microservices. If you haven’t done it yet, check it out using [ 15-days free trial! ][28]

 [1]: https://thenewstack.io/five-principles-monitoring-microservices/
 [2]: https://sysdigrp2rs.wpengine.com/blog/kubernetes-service-discovery-docker/
 [3]: https://kubernetes.io/docs/concepts/services-networking/ingress/
 [4]: https://linkerd.io/
 [5]: https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/
 [6]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/linkerd1.png
 [7]: https://kubernetes.io/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/
 [8]: https://gist.github.com/mateobur/714cf94cc42c65f3cc7a0f4b8730d4f2
 [9]: https://blog.buoyant.io/2016/10/04/a-service-mesh-for-kubernetes-part-i-top-line-service-metrics/
 [10]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/linkerd2.png
 [11]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/linkerd3.png
 [12]: https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/206770633
 [13]: https://sysdigrp2rs.wpengine.com/blog/how-to-collect-statsd-metrics-in-containers/
 [14]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/linkerd4.png
 [15]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/linkerd5.png
 [16]: https://linkerd.io/features/circuit-breaking/
 [17]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/linkerd6.png
 [18]: http://zipkin.io/
 [19]: #service-latency
 [20]: #connection-failure
 [21]: #available-endpoints
 [22]: #retries-budget
 [23]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/linkerd7.png
 [24]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/linkerd8.png
 [25]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/linkerd9.png
 [26]: https://sysdigrp2rs.wpengine.com/blog/alerting-kubernetes/
 [27]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/linkerd10.png
 [28]: https://sysdigrp2rs.wpengine.com/