---
ID: 6553
post_title: >
  How to monitor Istio, the Kubernetes
  service mesh
author: Mateo Burillo
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/monitor-istio/
published: true
post_date: 2018-02-27 12:36:39
---
In this article we are going to deploy and monitor Istio over a Kubernetes cluster. Istio is a microservice mesh platform that offers advanced routing, balancing, security and high availability features, plus Prometheus-style metrics for your services out of the box.

## What is Istio? {#whatisistio}

<a href="https://istio.io/docs/concepts/what-is-istio/overview.html" target="_blank">Istio</a> is a platform used to interconnect microservices. It provides advanced network features like load balancing, service-to-service authentication, monitoring, etc, without requiring any changes in service code.

In the Kubernetes context, Istio deploys an <a href="https://github.com/envoyproxy/envoy" target="_blank">Envoy proxy</a> as a sidecar container inside every pod that provides a service.

These proxies mediate every connection, and from that position they route the incoming / outgoing traffic and enforce the different security and network policies.

This dynamic group of proxies is managed by the Istio “control plane”, a separate set of pods that orchestrate the routing, security, live ruleset updates, etc.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_arch_overview.png" alt="Istio Sysdig Arch Overview" width="600" height="500" class="alignleft size-full wp-image-6561" />][1] 

You have detailed descriptions of each subsystem component in the <a href="https://istio.io/docs/concepts/what-is-istio/overview.html" target="_blank">Istio project docs</a>.

### Service mesh explained: The rise of the "service mesh" {#servicemeshexplainedtheriseoftheservicemesh}

Containers are incredibly light and fast, it's no surprise their density is roughly one order of magnitude greater than virtual machines. Classical monolithic component interconnection diagrams are rapidly turning into highly dynamic, fault tolerant, N-to-N communications with their own internal security rules, labeling-based routes, DNS and service directories, etc. The famous **microservice mesh**.

This means that while software autonomous units (containers) are becoming simpler and numerous, interconnection and troubleshooting distributed software behavior is actually getting harder.

And of course, we don't want to burden containers with this complexity, we want them to stay thin and platform agnostic.

Kubernetes already offers a basic abstraction layer separating the <a href="https://kubernetes.io/docs/concepts/services-networking/service/" target="_blank">service</a> itself from the server pods. Several software projects are striving to tame this complexity, offering visibility, traceability and other advanced pod networking features, we already covered how to monitor <a href="https://sysdigrp2rs.wpengine.com/blog/monitor-linkerd/" target="_blank">Linkerd</a>, let's now talk about Istio.

[tweet_box design="default" float="none"]Deploy and monitor #Istio in your #Kubernetes cluster for #microservice mesh discovery, security and advanced routing[/tweet_box] 

### Istio features overview {#istiofeaturesoverview}

*   <a href="https://istio.io/docs/concepts/traffic-management/overview.html" target="_blank">Intelligent routing and load balancing</a>: Policies to map static service interfaces to different backend versions, allowing for A/B testing, canary deployments, gradual migration, etc. Istio also allows you to define routing rules based on HTTP-layer metadata like session tokens or user agent string.
*   <a href="https://istio.io/docs/concepts/traffic-management/handling-failures.html" target="_blank">Network resilience and health checks</a>: timeouts, retry budgets, health checks and circuit breakers. Ensuring that unhealthy pods can be quickly weeded out of the service mesh
*   <a href="https://istio.io/docs/concepts/policy-and-control/mixer.html" target="_blank">Policy Enforcement</a>: Peer TLS authentication, pre-condition checking (whitelists and similar ACL), quota management to avoid service abuse and/or consumer starvation.
*   <a href="https://istio.io/docs/concepts/what-is-istio/overview.html" target="_blank">Telemetry, traceability and troubleshooting</a>: telemetry is automatically injected in any service pod providing Prometheus-style network and L7 protocol metrics, Istio also dynamically traces the flow and chained connections of your microservices mesh.

## How to deploy Istio in Kubernetes {#howtodeployistioinkubernetes}

### Istio deployment overview {#istiodeploymentoverview}

Istio developers have made deploying the platform in a new or existing Kubernetes cluster simple enough.

Just make sure that your Kubernetes version is 1.7.3 or newer, with RBAC enabled. And that you don't have any older version of Istio already installed on the system.

Following the <a href="https://istio.io/docs/setup/kubernetes/quick-start.html#installation-steps" target="_blank">installation instructions here</a>:

    curl -L https://git.io/getLatestIstio | sh -
    cd istio-<version>
    export PATH=$PWD/bin:$PATH

Now you need to decide whether or not you want mutual TLS authentication between pods. If you choose to enable TLS your Istio services won't be allowed to talk to non-Istio entities.

We will use the non-TLS version this time:

    kubectl apply -f install/kubernetes/istio.yaml

Istio system services and pods will be ready in a few minutes:

    $ kubectl get svc -n istio-system
    NAME            CLUSTER-IP      EXTERNAL-IP      PORT(S)                                                            AGE
    istio-ingress   10.15.240.67    some-external-ip   80:32633/TCP,443:31389/TCP                                         1d
    istio-mixer     10.15.249.77    <none>           9091/TCP,15004/TCP,9093/TCP,9094/TCP,9102/TCP,9125/UDP,42422/TCP   1d
    istio-pilot     10.15.243.227   <none>           15003/TCP,443/TCP                                                  1d
    
    $ kubectl get pod -n istio-system
    NAME                             READY     STATUS    RESTARTS   AGE
    istio-ca-1363003450-vp7pt        1/1       Running   0          1d
    istio-ingress-1005666339-w1gcs   1/1       Running   0          1d
    istio-mixer-465004155-nncrd      3/3       Running   0          1d
    istio-pilot-1861292947-zlt8w     2/2       Running   0          1d

### Injecting the Istio "Envoy" proxy in your existing Kubernetes pods {#injectingtheistioenvoyproxyinyourexistingkubernetespods}

As we mentioned in the architecture diagram, any service pod needs to be bundled with the Envoy container. Your Kubernetes cluster can be automatically instructed to do it if the alpha cluster features are enabled and you deploy the Istio-initializer:

    kubectl apply -f install/kubernetes/istio-initializer.yaml

Alternatively, you can rewrite your existing yaml definitions on the fly using *istioctl*

    kubectl create -f <(istioctl kube-inject -f <your-app-spec>.yaml)

Let's try with a simple single container deployment and service

    $ kubectl apply -f <(istioctl kube-inject -f flask.yaml)
    $ kubectl logs flask-1027288086-lr4fm
    a container name must be specified for pod flask-1027288086-lr4fm, choose one of: [flask istio-proxy]

Ok, it looks like our pods and services have been correctly instrumented.

## How to monitor Istio using Prometheus {#howtomonitoristiousingprometheus}

One of the major infrastructure enhancements of tunneling your service traffic through the Istio Envoy proxies is that you automatically collect metrics that are fine-grained and provide high level application information (since they are reported for every service proxy).

These individuals metrics are then forwarded to the <a href="https://istio.io/docs/concepts/policy-and-control/mixer.html" target="_blank">Mixer</a> component, which aggregates them for the entire mesh.

Mixer provides <a href="https://istio.io/docs/tasks/telemetry/querying-metrics.html" target="_blank">three Prometheus endpoints</a>:

1.  istio-mesh (istio-mixer.istio-system:42422): all Mixer-generated mesh metrics.
2.  mixer (istio-mixer.istio-system:9093): all Mixer-specific metrics. Used to monitor Mixer itself.
3.  envoy (istio-mixer.istio-system:9102): raw stats generated by Envoy (and translated from statsd to prometheus).

Istio project also provides examples and documentation on configuring a <a href="https://istio.io/docs/tasks/telemetry/querying-metrics.html" target="_blank">Prometheus</a> server to scrape and analyze the most relevant metrics.

    kubectl apply -f install/kubernetes/addons/prometheus.yaml

Wait until the pod is ready, and forward the Prometheus server port to your local machine

    kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=prometheus -o jsonpath='{.items[0].metadata.name}') 9090:9090 &

You can now access the Prometheus server UI opening http://localhost:9090/ in your web browser

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_prometheus_overview.png" alt="Istio Sysdig Prometheus Overview" width="800" height="300" class="alignleft size-full wp-image-6570" />][2] 

There is also a <a href="https://istio.io/docs/tasks/telemetry/using-istio-dashboard.html" target="_blank">Grafana deployment</a> already preconfigured and ready to test at the Istio repository:

    $ kubectl create -f install/kubernetes/addons/grafana.yaml

Again, wait for the pod and service to be up and running and redirect the Grafana service port

    kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=grafana -o jsonpath='{.items[0].metadata.name}') 3000:3000 &

You can access a pre populated Dashboard at <a href="http://localhost:3000/dashboard/db/istio-dashboard" target="_blank">http://localhost:3000/dashboard/db/istio-dashboard</a>

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_grafana_overview-1024x557.png" alt="Istio Sysdig Grafana Overview" width="1024" height="557" class="alignleft size-large wp-image-6566" />][3] 

### How to monitor Istio with Sysdig scraping Prometheus metrics {#howtomonitoristiowithsysdigscrapingprometheusmetrics}

Istio core services using the Prometheus metric format is very convenient, because as you know, <a href="https://sysdigrp2rs.wpengine.com/blog/prometheus-metrics/" target="_blank">Sysdig will automatically detect and scrape Prometheus endpoints.</a>

Let's edit <a href="https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/206443225-How-can-I-edit-the-agent-s-configuration-file-" target="_blank">Sysdig agent configuration file (dragent.yaml)</a> to configure which pods and ports should be scraped:

    prometheus:
      enabled: true
    …
       - include:
             kubernetes.pod.annotation.prometheus.io/scrape: true
             conf:
               port_filter:
                 - include: 42422
                 - include: 9102
                 - include: 9093

Make sure that Prometheus is enabled and then, write an *include* filter. For this example, we use <a href="https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/" target="_blank">Kubernetes annotations</a>, this way we can easily keep adding hosts without changing the agent configuration again.

Let's annotate the Mixer pod (your specific serial number will vary):

    kubectl annotate pod istio-mixer-465004155-nncrd -n istio-system prometheus.io/scrape=true

Logging into the Sysdig Monitor web console, we check that the new metrics are indeed flowing to our cloud platform (metricCount.prometheus).

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_scraping_prometheus-1024x336.png" alt="Istio Sysdig Scraping Prometheus" width="1024" height="336" class="alignleft size-large wp-image-6571" />][4] 

We are scraping istio-mixer Prometheus endpoints, time to monitor Istio!

## Monitoring Istio: reference metrics and dashboards {#monitoringistioreferencemetricsanddashboards}

Let's start from the beginning, monitoring our services and application behaviour.

Segmenting by service and service version, a few usual metrics that you want to monitor (and create the associated Dashboards):

*   Number of requests *istio_request_count*
*   Request duration *istio_request_duration.avg*, *istio_request_duration.count*
*   Request size *http_request_size_bytes.count*
*   Also, 90-99 percentiles of these metric, to delimit your worst case scenario
    *   *http_request_size_bytes.90percentile*
    *   *http_request_duration_microseconds.90percentile*
    *   *http_request_size_bytes.99percentile*
    *   *http_request_duration_microseconds.99percentile*
*   HTTP Error codes response_code
*   Bandwidth and disk IO consumption in your serving pods net.bytes.total file.iops.total
*   Top accessed URLs net.http.url, net.http.method

Using the Sysdig dashboard wizard you can quickly assemble your custom service overview with the most important metrics

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_dashboard_example-1024x589.png" alt="Istio Sysdig Dashboard Example" width="1024" height="589" class="alignleft size-large wp-image-6565" />][5] 

Or just use the brand new Istio default dashboards.

Istio System Overview

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_overview_dashboard-1024x548.png" alt="Istio Sysdig Dashboard Overview" width="1024" height="548" class="alignleft size-large wp-image-6587" />][6] 

Istio Service

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_service_dashboard-1024x548.png" alt="Istio Sysdig Service Dashboard" width="1024" height="548" class="alignleft size-large wp-image-6588" />][7] 

If you are familiar with Sysdig or have read other articles related to <a href="https://sysdigrp2rs.wpengine.com/blog/monitor-nginx-kubernetes/" target="_blank">Kubernetes and multiple HTTP services</a> you will soon realize than you already had similar HTTP / network metrics out of the box.

So, apart from the essentials, let's highlight some of the additional features that Istio brings to the table in terms of monitoring.

HTTP request and response size (in bytes), not just as in network bandwidth but measuring each HTTP connection individually:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_bandwidth_service-1024x382.png" alt="Istio Sysdig Bandwidth Service" width="1024" height="382" class="alignleft size-large wp-image-6562" />][8] 

You have specific metrics to monitor <a href="https://grpc.io/" target="_blank">gRPC</a> connections, segmented by method, code, service, type, etc. Istio is able to <a href="https://istio.io/docs/concepts/what-is-istio/overview.html#envoy" target="_blank">route HTTP/2 & gRPC</a> through its proxies.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_grpc-1024x298.png" alt="Istio Sysdig GRPC" width="1024" height="298" class="alignleft size-large wp-image-6567" />][9] 

Thanks to Istio connection traceability, you can also monitor the mentioned metrics (request count, duration, etc) not only from the destination but also from the source internal service (or version thereof):

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_source_service-1024x412.png" alt="Istio Sysdig Source Service" width="1024" height="412" class="alignleft size-large wp-image-6572" />][10] 

## Monitoring Istio internals {#monitoringistiointernals}

Apart from monitoring the services, you can use Istio and Sysdig aggregated metrics to monitor Istio internal services health and performance.

Istio provides its own <a href="https://istio.io/docs/tasks/traffic-management/ingress.html" target="_blank">Ingress controller</a>, this is a very relevant piece of our infrastructure to monitor. When your users are experiencing performance problems or errors, the edge router is one of the first points to check.

To assess the global health of your edge router connections you can display its connections table, global HTTP response codes, resource usage, number of request per service or URL, etc.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_connection_table-1024x311.png" alt="Istio Sysdig Connection Table" width="1024" height="311" class="alignleft size-large wp-image-6564" />][11] 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_connections_stats-1024x263.png" alt="Istio Sysdig Connections Stats" width="1024" height="263" class="alignleft size-large wp-image-6563" />][12] 

Istio's <a href="https://istio.io/docs/reference/config/mixer/adapters/" target="_blank">Mixer has several adapters</a> where it forward information you can use the *mixer_adapter_dispatch_count* metric segmented by adapter these connections

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_mixer_info-1024x448.png" alt="Istio Sysdig Mixer Info" width="1024" height="448" class="alignleft size-large wp-image-6568" />][13] 

Mixer will also be contacted by the services to retrieve authorization and preconditions info, you can monitor these connections (and the result code)

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_mixer_info2-1024x479.png" alt="Istio Sysdig Mixer Info 2" width="1024" height="479" class="alignleft size-large wp-image-6569" />][14] 

You can use the "adapter" metrics to monitor Istio's Mixer communication with the adapters (Prometheus, Kubernetes)

*   *mixer_adapter_dispatch_count*
*   *mixer_adapter_dispatch_duration.avg*
*   *mixer_adapter_dispatch_duration.count*

And the "resolve" metrics to monitor Istio's Mixer communication with the services

*   *mixer_config_resolve_actions.avg*
*   *mixer_config_resolve_actions.count*
*   *mixer_config_resolve_count*
*   *mixer_config_resolve_duration.avg*
*   *mixer_config_resolve_duration.count*
*   *mixer_config_resolve_rules.avg*
*   *mixer_config_resolve_rules.count*

## Monitoring Istio A/B deployments and canary deployments {#monitoringistioabdeploymentsandcanarydeployments}

One of Istio major features is the ability to establish intelligent routing based on service version.

The pods that provide the backend for a certain service will have different Kubernetes labels

    Labels:         app=reviews
                    pod-template-hash=3187719182
                    version=v3

These different backends are transparent to the consumer (service or final user) but Istio can take advantage of this information to perform:

*   Content-based routing: For example if the user-agent is a mobile phone, you can change the specific service that formats the final HTML template
*   A/B deployments: two similar versions of the service that you want to compare in production
*   Canary deployment: Experimental service version that will only be triggered by certain conditions (like some specific test users)
*   Traffic Shifting: Progressive migration to the new service version maintaining the old version fully functional

Aggregating Istio and Sysdig metrics you can supervise these service migration will all the information you need to take further decisions.

For example we are comparing the *alpha* and *beta* service pods, they provide the same Kubernetes service, using <a href="https://istio.io/docs/tasks/traffic-management/traffic-shifting.html" target="_blank">Istio traffic shifting</a>, we decide to split ingress traffic 50-50.

As you can see the number of requests and duration of requests (two top graphs) is extremely similar, so we can assume it's a fair comparison in terms of load.

If you look at the two bottom graphs, it turns out that service *alpha* is suffering almost 3 times the number of HTTP errors and also its worst case response time (99 percentile down-right graph) is also significatively higher than service *beta*. Looks like our developers did a great job with the new version :).

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_ab_deployment-1024x501.png" alt="Istio Sysdig AB Deployment" width="1024" height="501" class="alignleft size-large wp-image-6560" />][15] 

## Conclusions {#conclusions}

Istio solves the "mesh tangle" adding a transparent proxy as a sidecar to your service-provider pods. From this vantage position, it can collect fine-grained metrics and dynamically modify the routing flow without interfering with the pod software at all.

This strategy nicely complements Sysdig analogus non-intrusive, minimal-instrumentation approach to maintain your service pods simple and infrastructure agnostic (as they should be®).

The service mesh will be a hot topic for Kubernetes in 2018 and the jury is still out, we will keep an eye on the ecosystem to compare the family of service mesh infrastructures that are growing around the container stack.

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_arch_overview.png
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_prometheus_overview.png
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_grafana_overview.png
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_scraping_prometheus.png
 [5]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_dashboard_example.png
 [6]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_overview_dashboard.png
 [7]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_service_dashboard.png
 [8]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_bandwidth_service.png
 [9]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_grpc.png
 [10]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_source_service.png
 [11]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_connection_table.png
 [12]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_connections_stats.png
 [13]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_mixer_info.png
 [14]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_mixer_info2.png
 [15]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/istio_sysdig_ab_deployment.png