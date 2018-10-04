---
ID: 9692
post_title: >
  Kubernetes Monitoring with Prometheus
  -The ultimate guide (part 1).
author: Mateo Burillo
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/kubernetes-monitoring-prometheus/
published: true
post_date: 2018-08-16 01:29:24
---
Prometheus monitoring is fast becoming one of the Docker and Kubernetes monitoring tool to use. This guide explains how to implement Kubernetes monitoring with Prometheus. You will learn how to deploy Prometheus server, metrics exporters, setup kube-state-metrics, pull, scrape and collect metrics, configure alerts with Alertmanager and dashboards with Grafana. We'll cover how to do this manually as well as by leveraging some of the automated deployment/install methods like Prometheus operators.

This guide is comprised of four parts:

**1 -** (this one)

*   [Intro to Prometheus and its core concepts][1]
*   [How Prometheus compares to other monitoring solutions][2]
*   [How to install Prometheus][3]
*   [Monitoring a Kubernetes Service][4]
    *   [Prometheus exporters][5]
*   [Monitoring a Kubernetes cluster][6]
    *   [Kubernetes node][7]
    *   [Kube State Metrics][8]
*   [Kubernetes internal components][9] 

**2 -** [How to configure and run additional components of the Prometheus stack inside Kubernetes: Alertmanager, Push gateway, Grafana, external storage, alert receivers.][10]

**3 -** The Prometheus operator, Custom Resource Definitions, fully automated Kubernetes deployment for Prometheus, AlertManager and Grafana.

**4 -** Prometheus performance considerations, high availability, external storage, dimensionality limits. [tweet_box design="default" float="none"]Monitor your #Kubernetes cluster using #Prometheus, build the full stack covering Kubernetes cluster components, deployed microservices, alerts and dashboards (1/4).[/tweet_box] 

### Why Use Prometheus for Kubernetes Monitoring {#whyuseprometheusforkubernetesmonitoring}

Two technology shifts took place that created a need for a new monitoring framework:

*   **DevOps culture:** Prior to the emergence of DevOps, monitoring was comprised of hosts, networks and services. Now developers need the ability to easily integrate app and business related metrics as an organic part of the infrastructure, because they are more involved in the CI/CD pipeline and can do a lot of operations-debugging on their own. Monitoring needed to be democratized, made more accessible, and cover additional layers of the stack.
*   **Containers and Kubernetes:** Container-based infrastructures are radically changing how we do logging, debugging, high-availability… and monitoring is not an exception. Now you have a huge number of volatile software entities, services, virtual network addresses, exposed metrics that suddenly appear or vanish. Traditional monitoring tools are not designed to handle this.

Why Prometheus Is the Right Tool for Containerized Environments

*   **Multi-dimensional data model:** The model is based on <a href="https://prometheus.io/docs/concepts/data_model/" target="_blank" rel="noopener">key-value pairs</a>, similar to how Kubernetes itself organizes infrastructure metadata using labels. It allows for flexible and accurate time series data, powering its Prometheus query language.
*   **Accessible format and protocols:** Exposing prometheus metrics is a pretty straightforward task. Metrics are human readable, are in a self-explanatory format, and are published using a standard HTTP transport. You can check that the metrics are correctly exposed just using your web browser:

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/prom_kubernetes_metrics.png" alt="" width="1061" height="243" class="alignnone size-full wp-image-9694" />

Monitoring prometheus with sysdig is awesome right?

*   **Service discovery:** The Prometheus server is in charge of periodically scraping the targets, so that applications and services don't need to worry about emitting data (metrics are pulled, not pushed). These Prometheus servers have several methods to auto-discover scrape targets, some of them can be configured to filter and match container metadata, making it an excellent fit for ephemeral Kubernetes workloads.
*   **Modular and highly available components:** Metric collection, alerting, graphical visualization, etc, are performed by different composable services. All these services are designed to support redundancy and sharding.

### How Prometheus compares to other Kubernetes monitoring tools {#howprometheuscomparestootherkubernetesmonitoringtools}

Prometheus released version 1.0 during 2016, so it's a fairly recent technology. There were a wealth of tried-and-tested monitoring tools available when Prometheus first appeared. How does Prometheus compare with other veteran monitoring projects?

**Key-value vs dot-separated dimensions:** Several engines like StatsD/Graphite use an explicit <a href="https://sysdigrp2rs.wpengine.com/blog/prometheus-metrics/#prometheus-metrics-format-dot-metrics-vs-tagged-metrics" target="_blank" rel="noopener">dot-separated format to express dimensions</a>, effectively generating a new metric per label:

    current_active_users.free_tier = 423
    current_active_users.premium = 56

This method can become cumbersome when trying to expose highly dimensional data (containing lots of different labels per metric). Flexible query-based aggregation becomes more difficult as well.

Imagine that you have 10 servers and want to group by error code. Using key-value, you can simply group the flat metric by `{http_code="500"}`. Using dot-separated dimensions, you will have a big number of independent metrics that you need to aggregate using expressions.

**Event logging vs metrics recording:** InfluxDB / Kapacitor are more similar to the Prometheus stack. They use label-based dimensionality and the same data compression algorithms. Influx is, however, more suitable for event logging due to its nanosecond time resolution and ability to merge different event logs. Prometheus is more suitable for metrics collection and has a more powerful query language to inspect them.

**Blackbox vs whitebox monitoring:** As we mentioned before, tools like Nagios/Icinga/Sensu are suitable for host/network/service monitoring, classical sysadmin tasks. Nagios, for example, is host-based. If you want to get internal detail about the state of your micro-services (aka <a href="https://insights.sei.cmu.edu/devops/2016/08/whitebox-monitoring-with-prometheus.html" target="_blank" rel="noopener">whitebox monitoring</a>), Prometheus is a more appropriate tool.

## The challenges of microservices and Kubernetes monitoring with Prometheus {#thechallengesofmicroservicesandkubernetesmonitoring}

There are unique challenges unique to monitoring a Kubernetes cluster(s) that need to be solved for in order to deploy a reliable monitoring / alerting / graphing architecture.

### Monitoring containers: visibility {#monitoringcontainersvisibility}

Containers are lightweight, mostly immutable black boxes, which can present monitoring challenges… The Kubernetes API and the <a href="https://github.com/kubernetes/kube-state-metrics" target="_blank" rel="noopener">kube-state-metrics</a> (which natively uses prometheus metrics) solve part of this problem by exposing Kubernetes internal data such as number of desired / running replicas in a deployment, unschedulable nodes, etc.

Prometheus is a good fit for microservices because you just need to expose a metrics port, and thus don't need to add too much complexity or run additional services. Often, the service itself is already presenting a HTTP interface, and the developer just needs to add an additional path like `/metrics`.

In some cases, the service is not prepared to serve Prometheus metrics and you can't modify the code to support it. In that case, you need to deploy a <a href="https://prometheus.io/docs/instrumenting/exporters/" target="_blank" rel="noopener">Prometheus exporter</a> bundled with the service, often as a sidecar container of the same pod.

### Dynamic monitoring: changing and volatile infrastructure {#dynamicmonitoringchangingandvolatileinfrastructure}

As we mentioned before, ephemeral entities that can start or stop reporting any time are a problem for classical, more static monitoring systems.

Prometheus has <a href="https://prometheus.io/docs/prometheus/latest/configuration/configuration" target="_blank" rel="noopener">several autodiscover mechanisms</a> to deal with this. The most relevant for this guide are:

**Consul:** A tool for service discovery and configuration. Consul is distributed, highly available, and extremely scalable.

**Kubernetes:** Kubernetes SD configurations allow retrieving scrape targets from Kubernetes' REST API and always staying synchronized with the cluster state.

**Prometheus Operator:** To automatically generate monitoring target configurations based on familiar Kubernetes label queries. We will focus on this deployment option later on.

### Monitoring new layers of infrastructure: Kubernetes components {#monitoringnewlayersofinfrastructurekubernetescomponents}

Using Kubernetes concepts like the physical host or service port become less relevant. You need to organize monitoring around different groupings like microservice performance (with different pods scattered around multiple nodes), namespace, deployment versions, etc.

Using the label-based data model of Prometheus together with the <a href="https://prometheus.io/docs/prometheus/latest/querying/basics/" target="_blank" rel="noopener">PromQL</a>, you can easily adapt to these new scopes.

## Kubernetes monitoring with Prometheus: Architecture overview {#kubernetesmonitoringwithprometheusarchitectureoverview}

We will get into more detail later on, this diagram covers the basic entities we want to deploy in our Kubernetes cluster:

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/prometheus_kubernetes_diagram_overview.png" alt="" width="960" height="547" class="alignnone size-full wp-image-9695" />

*   1 - The Prometheus servers need as much target auto discovery as possible.
    *   There are several options to achieve this:
        *   Consul
        *   Prometheus Kubernetes SD plugin
        *   The Prometheus operator and its <a href="https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/" target="_blank" rel="noopener">Custom Resource Definitions</a>
*   2 - Apart from application metrics, we want Prometheus to collect metrics related to the Kubernetes services, nodes and orchestration status.
    *   <a href="https://github.com/prometheus/node_exporter" target="_blank" rel="noopener">Node exporter</a>, for the classical host-related metrics: cpu, mem, network, etc.
    *   Kube-state-metrics for orchestration and cluster level metrics: deployments, pod metrics, resource reservation, etc.
    *   Kube-system metrics from internal components: kubelet, etcd, dns, scheduler, etc.
*   3 - Prometheus can configure rules to trigger alerts using PromQL, alertmanager will be in charge of managing alert notification, grouping, inhibition, etc.
*   4 - The alertmanager component configures the receivers, gateways to deliver alert notifications.
*   5 - Grafana can pull metrics from any number of Prometheus servers and display panels and Dashboards.

## How to install Prometheus {#howtoinstallprometheus}

There are different ways to install Prometheus in your host or in your Kubernetes cluster:

*   Directly as a single binary running on your hosts, which is fine for learning, testing and developing purposes but not appropriate for a containerized deployment.
*   As a Docker container which has, in turn, several orchestration options:
    *   Raw Docker containers, Kubernetes Deployments / StatefulSets, the Helm Kubernetes package manager, Kubernetes operators, etc.

Let's get from more manual to more automated deployments:

Single binary -> Docker container -> Kubernetes Deployment -> Prometheus operator (Chapter 3)

You can directly <a href="https://prometheus.io/download/" target="_blank" rel="noopener">download and run</a> the Prometheus binary in your host:

    prometheus-2.3.1.linux-amd64$ ./prometheus
    level=info ts=2018-06-21T11:26:21.472233744Z caller=main.go:222 msg="Starting Prometheus"

Which may be nice to get a first impression of the Prometheus web interface (port 9090 by default).

A better option is to deploy the Prometheus server inside a container:

    docker run -p 9090:9090 -v /tmp/prometheus.yml:/etc/prometheus/prometheus.yml \
           prom/prometheus

Note that you can easily adapt this Docker container into a proper Kubernetes Deployment object that will mount the configuration from a ConfigMap, expose a service, deploy multiple replicas, etc.

    kubectl create configmap prometheus-example-cm --from-file prometheus.yml

(You have a basic working prometheus.yml config file <a href="https://github.com/prometheus/prometheus/blob/master/docs/getting_started.md" target="_blank" rel="noopener">here</a>)

And then you can apply this example yaml:

<script src="https://gist.github.com/aa9ce08ca331cfa686874567a093e557.js"></script> <noscript>
  <pre><code>
File: prometheus-example.yaml
-----------------------------

apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus-deployment
  labels:
    app: prometheus
    purpose: example
spec:
  replicas: 2
  selector:
    matchLabels:
      app: prometheus
      purpose: example
  template:
    metadata:
      labels:
        app: prometheus
        purpose: example
    spec:
      containers:
      - name: prometheus-example
        image: prom/prometheus
        volumeMounts:
          - name: config-volume
            mountPath: /etc/prometheus/prometheus.yml
            subPath: prometheus.yml
        ports:
        - containerPort: 9090
      volumes:
        - name: config-volume
          configMap:
           name: prometheus-example-cm
---
kind: Service
apiVersion: v1
metadata:
  name: prometheus-example-service
spec:
  selector:
    app: prometheus
    purpose: example
  ports:
  - name: promui
    protocol: TCP
    port: 9090
    targetPort: 9090

</code></pre>
</noscript>

If you don't want to configure a LoadBalancer / external IP, then you can always specify the type <a href="https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types" target="_blank" rel="noopener">NodePort</a> for your service.

After a few seconds, you should see the Prometheus pods running in your cluster:

    kubectl get pods
    NAME                                     READY     STATUS    RESTARTS   AGE
    prometheus-deployment-68c5f4d474-cn5cb   1/1       Running   0          3h
    prometheus-deployment-68c5f4d474-ldk9p   1/1       Running   0          3h

There are several configuration tweaks that you can implement at this point, such as configuring <a href="https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity" target="_blank" rel="noopener">pod antiaffinity</a> to force the Prometheus server pods to be located in different nodes. We will cover the performance and high availability aspects on the fourth chapter of this guide.

A more advanced and automated option is to use the <a href="https://github.com/coreos/prometheus-operator" target="_blank" rel="noopener">Prometheus operator</a>, covered in the third chapter of this guide. You can think of it as a meta-deployment: a deployment that manages other deployments and configures and updates them according to high-level service specifications. We will start manually configuring the Prometheus stack and in the next chapter we will use the operator to make the Prometheus deployment easily portable, declarative and flexible.

## How to monitor a Kubernetes service with Prometheus {#howtomonitorakubernetesservicewithprometheus}

Prometheus metrics are exposed by services through HTTP(S), and there are several advantages of this approach compared to other similar monitoring solutions:

*   You don't need to install a service agent, just expose a web port. Prometheus servers will regularly scrape (pull), so you don't need to worry about pushing metrics or configuring a remote endpoint either.
*   Several microservices already use HTTP for their regular functionality, and you can reuse that internal webserver and just add a folder like `/metrics`.
*   The metrics format itself is human-readable and easy to grasp. If you are the maintainer of the microservice code, you can start publishing metrics without much complexity or learning required.

Some services are designed to expose Prometheus metrics from the ground-up (the Kubernetes kubelet, Traefik web proxy, Istio microservice mesh, etc). Some other services are not natively integrated, but can be easily adapted using an exporter. An exporter is a service that collects service stats and "translates" to Prometheus metrics ready to be scraped. There are examples of both in this chapter.

Let's start with the best case scenario: the microservice that you are deploying already offers a Prometheus endpoint.

<a href="https://traefik.io/" target="_blank" rel="noopener">Traefik</a> is a reverse proxy designed to be tightly integrated with microservices and containers. A common use case for Traefik is to be used as an Ingress controller or Entrypoint, this is, the bridge between Internet and the specific microservices inside your cluster.

You have several options to <a href="https://docs.traefik.io/" target="_blank" rel="noopener">install Traefik</a> and a <a href="https://github.com/containous/traefik/blob/master/docs/user-guide/kubernetes.md" target="_blank" rel="noopener">Kubernetes-specific install guide</a>. If you just want a simple Traefik deployment with Prometheus support up and running quickly, use the following snippet:

    kubectl create -f https://raw.githubusercontent.com/mateobur/prometheus-monitoring-guide/master/traefik-prom.yaml

Once the Traefik pods is running and you can display the service IP:

    kubectl get svc
    NAME                         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                   AGE
    kubernetes                   ClusterIP   10.96.0.1       <none>        443/TCP                   238d
    prometheus-example-service   ClusterIP   10.103.108.86   <none>        9090/TCP                  5h
    traefik                      ClusterIP   10.108.71.155   <none>        80/TCP,443/TCP,8080/TCP   35s

You can check that the Prometheus metrics are being exposed just using curl:

    curl 10.108.71.155:8080/metrics
    # HELP go_gc_duration_seconds A summary of the GC invocation durations.
    # TYPE go_gc_duration_seconds summary
    go_gc_duration_seconds{quantile="0"} 2.4895e-05
    go_gc_duration_seconds{quantile="0.25"} 4.4988e-05
    ...

Now, you need to add the new target to the `prometheus.yml` conf file.

You will notice that Prometheus automatically scrapes itself:

      - job_name: 'prometheus'
    
        # metrics_path defaults to '/metrics'
        # scheme defaults to 'http'.
    
        static_configs:
        - targets: ['localhost:9090']

Let's add another static endpoint:

      - job_name: 'traefik'
        static_configs:
        - targets: ['traefik:8080']

If the service is in a different namespace you need to use the FQDN (i.e. `traefik.default.svc.cluster.local`)

Of course, this is a bare-minimum configuration, the <a href="https://prometheus.io/docs/prometheus/latest/configuration/configuration/#%3Cscrape_config%3E" target="_blank" rel="noopener">scrape config</a> supports multiple parameters.

To name a few:

*   `basic_auth` and `bearer_token`: You endpoints may require authentication over HTTPS, using a classical login/password scheme or a bearer token in the request headers.
*   `kubernetes_sd_configs` or `consul_sd_configs`: different endpoint autodiscovery methods.
*   `scrape_interval`, `scrape_limit`, `scrape_timeout`: Different tradeoffs between precision, resilience and system load.

We will use some of these in the next chapters, as the deployed Prometheus stack gets more complete.

Patch the ConfigMap and Deployment:

    kubectl create configmap prometheus-example-cm --from-file=prometheus.yml -o yaml --dry-run | kubectl apply -f -
    
    kubectl patch deployment prometheus-deployment -p \
      "{\"spec\":{\"template\":{\"metadata\":{\"labels\":{\"date\":\"`date +'%s'`\"}}}}}"

If you access the `/targets` URL in the Prometheus web interface, you should see the Traefik endpoint UP:

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/prometheus_kubernetes_traefik.png" alt="" width="1439" height="674" class="alignnone size-full wp-image-9698" />

Using the main web interface, we can locate some traefik metrics (very few of them, because we don't have any Traefik frontends or backends configured for this example) and retrieve its values:

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/prometheus_kubernetes_traefik-2-1.png" alt="" width="1058" height="704" class="alignnone size-full wp-image-9697" />  


We already have a Prometheus on Kubernetes working example!

## How to monitor Kubernetes services with Prometheus exporters Monitoring apps using Prometheus exporters {#howtomonitorkubernetesserviceswithprometheusexportersmonitoringappsusingprometheusexporters}

It's likely that many of the applications you want to deploy in your Kubernetes cluster do not expose Prometheus metrics out of the box. In that case, you need to bundle a <a href="https://prometheus.io/docs/instrumenting/exporters/" target="_blank" rel="noopener">Prometheus exporter</a>, an additional process that is able to retrieve the state / logs / other metric formats of the main service and expose this information as Prometheus metrics. In other words, a Prometheus adapter.

You can deploy a pod containing the Redis server and a Prometheus sidecar container with the following command:

    # Clone the repo if you don't have it already
    git clone git@github.com:mateobur/prometheus-monitoring-guide.git
    kubectl create -f prometheus-monitoring-guide/redis_prometheus_exporter.yaml

If you display the redis pod, you will notice it has two containers inside:

    kubectl get pod redis-546f6c4c9c-lmf6z
    NAME                     READY     STATUS    RESTARTS   AGE
    redis-546f6c4c9c-lmf6z   2/2       Running   0          2m

Now, you just need to update the Prometheus configuration and reload like we did in the last section:

      - job_name: 'redis'
        static_configs:
          - targets: ['redis:9121']

To obtain all the redis service metrics:

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/prometheus_kubernetes_redis-1.png" alt="" width="1808" height="1125" class="alignnone size-full wp-image-9696" />  


How to monitor Kubernetes applications with Prometheus

## Monitoring Kubernetes cluster with Prometheus and kube-state-metrics {#monitoringkubernetesclusterwithprometheusandkubestatemetrics}

In addition to monitoring the services deployed in the cluster, you also want to monitor the Kubernetes cluster itself. Three aspects of cluster monitoring to consider are:

*   The Kubernetes hosts (nodes) - classical sysadmin metrics such as cpu, load, disk, memory, etc.
*   Orchestration level metrics - Deployment state, resource requests, scheduling and api server latency, etc.
*   Internal kube-system components - Detailed service metrics for the scheduler, controller manager, dns service, etc.

The Kubernetes internal monitoring architecture has experienced some changes recently that we will try to summarize here, for more information you can read its <a href="https://github.com/kubernetes/community/blob/master/contributors/design-proposals/instrumentation/monitoring_architecture.md" target="_blank" rel="noopener">design proposal</a>.

### Monitoring Kubernetes components on a Prometheus stack {#kubernetesmonitoringcomponentsonaprometheusstack}

**Heapster:** Heapster is a cluster-wide aggregator of monitoring and event data that runs as a pod in the cluster.

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/kubernetes_monitoring_heapster.png" alt="" width="640" height="263" class="alignnone size-full wp-image-9693" />  


Apart from the Kubelets/cAdvisor endpoints, you can append additional metrics sources to Heapster like kube-state-metrics (see below).

Heapster is now **DEPRECATED**, its replacement is the metrics-server.

**cAdvisor:** <a href="https://github.com/google/cadvisor" target="_blank" rel="noopener">cAdvisor</a> is an open source container resource usage and performance analysis agent. It is purpose-built for containers and supports Docker containers natively. In Kubernetes, cAdvisor runs as part of the Kubelet binary, any aggregator retrieving node local and Docker metrics will directly scrape the Kubelet Prometheus endpoints.

**Kube-state-metrics:** <a href="https://sysdigrp2rs.wpengine.com/blog/introducing-kube-state-metrics/" target="_blank" rel="noopener">kube-state-metrics</a> is a simple service that listens to the Kubernetes API server and generates metrics about the state of the objects such as deployments, nodes and pods. It is important to note that kube-state-metrics is just a metrics endpoint, other entity needs to scrape it and provide long term storage (i.e. the Prometheus server).

**Metrics-server:** Metrics Server is a cluster-wide aggregator of resource usage data. It is intended to be the default Heapster replacement. Again, the metrics server will only present the last datapoints and it's not in charge of long term storage.

Thus:

*   Kube-state metrics is focused on orchestration metadata: deployment, pod, replica status, etc.
*   Metrics-server is focused on implementing the <a href="https://github.com/kubernetes/community/blob/master/contributors/design-proposals/instrumentation/resource-metrics-api.md" target="_blank" rel="noopener">resource metrics API</a>: CPU, file descriptors, memory, request latencies, etc.

### Monitoring the Kubernetes nodes with Prometheus {#monitoringthekubernetesnodeswithprometheus}

The Kubernetes nodes or hosts need to be monitored, we have plenty of tools to monitor a Linux host. In this guide we are going to use the Prometheus <a href="https://github.com/prometheus/node_exporter" target="_blank" rel="noopener">node-exporter</a>:

*   Its hosted by the Prometheus project itself
*   Is the one that will be automatically deployed when we use the Prometheus operator in the next chapters
*   Can be deployed as a DaemonSet and thus, will automatically scale if you add or remove nodes from your cluster.

You have several options to deploy this service, for example, using the DamonSet in this repo:

    kubectl create ns monitoring 
    kubectl create -f https://raw.githubusercontent.com/bakins/minikube-prometheus-demo/master/node-exporter-daemonset.yml

Or using <a href="https://docs.helm.sh/using_helm/#installing-helm" target="_blank" rel="noopener">Helm / Tiller</a>:

If you want to use Helm, remember to create the <a href="https://docs.helm.sh/using_helm/#role-based-access-control" target="_blank" rel="noopener">RBAC roles and service accounts</a> for the tiller component before proceeding.

    helm init --service-account tiller
    helm install --name node-exporter stable/prometheus-node-exporter

Once the chart is installed and running, you can display the service that you need to scrape:

    kubectl get svc 
    NAME                                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                                     AGE
    node-exporter-prometheus-node-exporter   ClusterIP   10.101.57.207    <none>        9100/TCP                                    17m

Once you add the scrape config like we did in the previous sections, you can start collecting and displaying the node metrics:

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/prometheus_monitoring_kube_node-1.png" alt="" width="1899" height="1183" class="alignnone size-full wp-image-9699" />  


### Monitoring kube-state-metrics with Prometheus {#monitoringkubestatemetricswithprometheus}

Deploying and monitoring the kube-state-metrics is also a pretty straightforward task. Again, you can deploy directly like in the example below or use a Helm chart.

    git clone https://github.com/kubernetes/kube-state-metrics.git
    kubectl apply -f kube-state-metrics/kubernetes/
    ...
    kubectl get svc -n kube-system
    NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
    kube-dns             ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP       13h
    kube-state-metrics   ClusterIP   10.102.12.190    <none>        8080/TCP,8081/TCP   1h

Again, you just need to scrape that service (port 8080) in the Prometheus config. Remember to use the FQDN this time:

      - job_name: 'kube-state-metrics'
        static_configs:
        - targets: ['kube-state-metrics.kube-system.svc.cluster.local:8080']

### Monitoring Kubernetes internal components with Prometheus {#monitoringkubernetesinternalcomponentswithprometheus}

There are several Kubernetes components including etcd, kube-scheduler or kube-controller that can expose its internal performance metrics using Prometheus.

Monitoring them is quite similar to monitoring any other Prometheus endpoint with two particularities:

*   Which network interfaces are these processes using to listen, http scheme and security (HTTP, HTTPS, RBAC) depend on your deployment method and configuration templates.
    *   Frequently, these services are only listening at localhost in the hosting node, making them difficult to reach from the Prometheus pods.
*   These components may not have a Kubernetes service pointing to the pods, but you can always create it.

Depending on your deployment method and configuration, the Kubernetes services may be listening on the local host only, to make things easier on this example we are going to use <a href="https://kubernetes.io/docs/setup/minikube/" target="_blank" rel="noopener">minikube</a>.

Minikube let's you spawn a local single-node Kubernetes virtual machine in minutes.

This will work as well on your hosted cluster, GKE, AWS etc, but you will need to reach the service port either by modifying the configuration and restarting the services or providing additional network routes.

Installing minikube is a fairly <a href="https://kubernetes.io/docs/tasks/tools/install-minikube/" target="_blank" rel="noopener">straightforward process</a>. First install the binary, then create a cluster that exposes the kube-scheduler service on all interfaces:

    minikube start --memory=4096 --bootstrapper=kubeadm --extra-config=kubelet.authentication-token-webhook=true --extra-config=kubelet.authorization-mode=Webhook --extra-config=scheduler.address=0.0.0.0 --extra-config=controller-manager.address=0.0.0.0

Create a service that will point to the kube-scheduler pod:

    kind: Service
    apiVersion: v1
    metadata:
      name: scheduler-service
      namespace: kube-system
    spec:
      selector:
        component: kube-scheduler
      ports:
      - name: scheduler
        protocol: TCP
        port: 10251
        targetPort: 10251

Now you will be able to scrape the endpoint: `scheduler-service.kube-system.svc.cluster.local:10251`

## What's next? {#whatsnext}

We already have a Prometheus deployment with 6 target endpoints: 2 "end-user" apps, 3 Kubernetes cluster endpoints and Prometheus itself. At this point the configuration is still naive and not automated, but we have a running Prometheus infrastructure.

The next chapter will cover additional components that are typically deployed together with the Prometheus service. We will start using the PromQL language to aggregate metrics, fire alerts and generate visualization dashboards.

 [1]: #whyuseprometheusforkubernetesmonitoring
 [2]: #howprometheuscomparestootherkubernetesmonitoringtools
 [3]: #howtoinstallprometheus
 [4]: #howtomonitorakubernetesservicewithprometheus
 [5]: #howtomonitorkubernetesserviceswithprometheusexportersmonitoringappsusingprometheusexporters
 [6]: #monitoringkubernetesclusterwithprometheusandkubestatemetrics
 [7]: #monitoringthekubernetesnodeswithprometheus
 [8]: #monitoringkubestatemetricswithprometheus
 [9]: #monitoringkubernetesinternalcomponentswithprometheus
 [10]: https://sysdigrp2rs.wpengine.com/blog/kubernetes-monitoring-with-prometheus-alertmanager-grafana-pushgateway-part-2/
