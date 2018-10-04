---
ID: 9820
post_title: 'Kubernetes Monitoring with Prometheus: AlertManager, Grafana, PushGateway (part 2).'
author: Mateo Burillo
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/kubernetes-monitoring-with-prometheus-alertmanager-grafana-pushgateway-part-2/
published: true
post_date: 2018-08-27 10:44:16
---
A complete 'Kubernertes monitoring with Prometheus' stack is comprised of much more than Prometheus servers that collect metrics by scraping endpoints. To deploy a real Kubernetes and microservices monitoring solution, you need many other supporting components including rules and alerts (AlertManager), a graphics visualization layer (Grafana), long term metrics storage, as well as extra metrics adapters for the software that is not compatible out of the box.

In this second part we are going to briefly cover all these supporting components, assuming that you already understand the basics of deploying a Prometheus monitoring server covered in the former chapter.

This guide is comprised of four parts:

1 – <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-monitoring-prometheus/" target="_blank">Kubernetes Monitoring with Prometheus, basic concepts and initial deployment</a>

2 - Kubernetes Monitoring with Prometheus: AlertManager, Grafana, PushGateway (this one):

*   [AlertManager, Prometheus alerting for Kubernetes][1]
*   [Grafana, Prometheus Dashboards][2]
*   [Prometheus metrics for ephemeral jobs - Push Gateway][3]
*   [Prometheus Persistent metrics storage][4]

3 - The Prometheus operator, Custom Resource Definitions, fully automated Kubernetes deployment for Prometheus, AlertManager and Grafana.

4 - Prometheus performance considerations, high availability, external storage, dimensionality limits. [tweet_box design="default" float="none"]Complete your #Kubernetes monitoring with Prometheus stack deploying #AlertManager, #Grafana, pushgateway and persistent storage.[/tweet_box] 

## Prometheus monitoring stack - Architecture overview {#prometheusmonitoringstackarchitectureoverview}

Let's start with a deployment architecture overview to place all the components we will discuss over the next sections.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/kubernetes_prom_diagram2.png" alt="" width="960" height="720" class="alignnone size-full wp-image-9821" />][5] 

1.  The **Prometheus servers**, which were explained in part 1, are at the core of this deployment. The Prometheus servers will push alerts to the **AlertManager** component, the Alertmanager will classify, route and notify using the different notification channels or receivers.
2.  We will configure a Prometheus datasource for **Grafana**, presenting data visualizations and Dashboard via its web interface.
3.  Using Kubernetes PersistentVolumes, we will configure long term metrics storage.
4.  We will also cover ephemeral maintenance tasks and its associated metrics. The **Pushgateway** will be in charge of storing them long enough to be collected by the Prometheus servers.

## AlertManager, Prometheus alerting for Kubernetes {#alertmanagerprometheusalertingforkubernetes}

There are two parts to alerting with Prometheus:

*   The actual alert conditions are configured using PromQL in the Prometheus servers.
*   The AlertManager component receives the active alerts:
    *   AlertManager classifies and groups them based on their metadata (labels), and optionally mutes or notifies them using a **receiver** (webhook, email, PagerDuty, etc etc).
    *   AlertManager is designed to be horizontally scaled, an instance can communicate with its peers providing minimal configuration.

We will start with some basic concepts, next section contains a practical example that you can run in your cluster straight away.

Let's review the structure of a Prometheus alerting rule:

    groups:
    - name: etcd
      rules:
      - alert: NoLeader
        expr: etcd_server_has_leader{job="kube-etcd"} == 0
        for: 1m
        labels:
          severity: critical
          k8s-component: etcd
        annotations:
          description: etcd member {{ $labels.instance }} has no leader
          summary: etcd member has no leader

*   The `expr` key is a Prometheus expression that will be periodically evaluated and will fire if true.
*   You can define a minimum evaluation time to avoid alerting on temporary, self-healing glitches.
*   Like many other designs in a Kubernetes context, the labels you choose are very relevant to grouping, classification and hierarchy. Later on, the AlertManager will take decisions about which alerts have higher priority, how to group alerts together, etc, all based on these labels.

Now you can display the alerts that the Prometheus server has successfully loaded, directly on the web interface (`Status -> Rules`):

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/kubernetes_prom_alert1.png" alt="" width="1332" height="618" class="alignnone size-full wp-image-9822" />][6] 

Also the ones that are firing right now (`Alerts`):

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/kubernetes_prom_alert2.png" alt="" width="1325" height="548" class="alignnone size-full wp-image-9823" />][7] 

Now you have some alert conditions and alerts that you need to forward to an AlertManager.

Like metrics endpoints, AlertManager services can also be <a href="https://prometheus.io/docs/prometheus/latest/configuration/configuration/#%3Calertmanager_config" target="_blank">autodetected</a> using different methods: DNS discovery, Consul, etc…

Given that we are talking about Prometheus monitoring in the Kubernetes context, we can take advantage of a basic Kubernetes abstraction: the service.

    alerting:
      alertmanagers:
      - scheme: http
        static_configs:
        - targets:
           - "alertmanager-service:9093"

From the point of view of the Prometheus server, this configuration is just a static name, but the Kubernetes service can perform different HA / LoadBalancing forwardings under the hood.

The <a href="https://prometheus.io/docs/alerting/alertmanager/" target="_blank">AlertManager</a> itself is a sophisticated piece of software, to cover the basics:

*   The AlertManager groups the different alerts based on their labels and origin
    *   This grouping and hierarchy form the "routing tree". A decision tree that determines which actions to take.
        *   For example, you can configure your routing tree so every alert with the label `k8s-cluster-component` gets mailed to the "cluster-admin" mail address.
*   Using Inhibition rules, an alert or group of alerts can be inhibited if another alert is firing. For example if a cluster is down and completely unreachable, then there is no point notifying the status of the individual microservices it contains.
*   Alerts can be forwarded to 'receivers', this is, notification gateways like email, PagerDuty, webhook, etc.

A simple example AlertManager config:

    global:
      resolve_timeout: 5m
    route:
      group_by: ['alertname']
      group_wait: 10s
      group_interval: 10s
      repeat_interval: 1h
      receiver: 'sysdig-test'
    receivers:
      - name: 'sysdig-test'
        webhook_configs:
        - url: 'https://webhook.site/8ce276c4-40b5-4531-b4cf-5490d6ed83ae'

This routing tree just configures a root node.

In this example, we are using a generic JSON webhook as a receiver, there is no need to deploy you own server <a href="https://webhook.site" target="_blank">https://webhook.site</a> will provide you with a temporary endpoint for testing purposes (obviously, your specific URL code for the example will vary).

### Prometheus monitoring with AlertManager, Try it out! {#prometheusmonitoringwithalertmanagertryitout}

You just need to clone the repository and apply the yaml files in the correct order:

    git clone git@github.com:mateobur/prometheus-monitoring-guide.git
    cd prometheus-monitoring-guide/alertmanager-example/
    kubectl create cm prometheus-example-cm --from-file prometheus.yml
    kubectl create cm prometheus-rules-general --from-file generalrules.yaml

Now, access <a href="https://webhook.site/" target="_blank">https://webhook.site/</a> and replace the random URL you will get in the `alertmanager.yml` file, `url: 'your url here'` parameter, and then:

    kubectl create cm alertmanager-config --from-file alertmanager.yml
    kubectl create -f prometheus-example.yaml
    kubectl create -f alertmanager-deployment.yaml

After a few seconds, all the pods should be in Running state:

    kubectl get pods
    NAME                                       READY     STATUS    RESTARTS   AGE
    alertmanager-deployment-78944d4bfc-rk2sg   1/1       Running   0          27s
    prometheus-deployment-784594c89f-fjbd6     1/1       Running   0          32s
    prometheus-deployment-784594c89f-gpvzf     1/1       Running   0          32s

This configuration includes a "DeadManSwitch", this is, an alert that will always fire, intended to test that all the microservices `Prometheus -> AlertManager -> Notification channels` are working as intended:

Check the webhook URL, looking for the notification in JSON format:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/kubernetes_prom_webhook.png" alt="" width="1173" height="884" class="alignnone size-full wp-image-9824" />][8] 

## Grafana, Kubernetes Monitoring with Prometheus - Dashboards {#grafanakubernetesmonitoringprometheusdashboards}

The <a href="https://grafana.com/" target="_blank">Grafana project</a> is an agnostic analytics and monitoring platform. It is not affiliated with Prometheus, but has become one of the most popular add-on components for creating a complete Prometheus solution.

We are going to use the <a href="https://helm.sh/" target="_blank">Helm Kubernetes package manager</a> for this deployment, which will enable us to ] configure Grafana using a set of repeatable ConfigMaps, avoiding any manual post-configuration:

    helm install --name mygrafana stable/grafana --set sidecar.datasources.enabled=true --set sidecar.dashboards.enabled=true --set sidecar.datasources.label=grafana_datasource --set sidecar.dashboards.label=grafana_dashboard

Pay attention to the instructions that helm will output by console, especially those for how to retrieve your admin password:

    kubectl get secret --namespace default mygrafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

You can directly forward the grafana service port to your local host to access the interface:

    kubectl port-forward mygrafana-859c68577f-4h6zp 3000:3000

Go to <a href="http://localhost:3000" target="_blank">http://localhost:3000</a> using your browser if you want to have a look at the interface:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/kubernetes_prom_grafana.png" alt="" width="1218" height="772" class="alignnone size-full wp-image-9825" />][9] 

Two configurations are required for Grafana to be useful:

*   Data sources, the Prometheus server in our case

*   The Dashboards to visualize your chosen metrics
    
    Conveniently, you can (and should, if possible) autoconfigure both from config files:

    kubectl create -f prometheus-monitoring-guide/grafana/
    configmap "dashboard-k8s-capacity" created
    configmap "sample-grafana-datasource" created

Here again, we are using the Service abstraction to point at several Prometheus server nodes with just one url

    url: http://prometheus-example-service:9090

And then Grafana will provide the visualization and Dashboards for the metrics collected by Prometheus:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/kubernetes_prom_grafana2.png" alt="" width="1778" height="906" class="alignnone size-full wp-image-9826" />][10] 

## Prometheus metrics for ephemeral jobs - Push Gateway {#prometheusmetricsforephemeraljobspushgateway}

Not every program is a continuously running service that you can expect to scrap at any random time. Every IT deployment has a myriad of one-off or cron tasks for backup, clean-up, maintenance, testing, etc.

You may want to collect some metrics from these tasks, but the target/scrap model of Prometheus is definitely not going to work here.

That's why the Prometheus project provides the <a href="https://github.com/prometheus/pushgateway" target="_blank">Pushgateway service</a>: push acceptor for ephemeral and batch jobs. You can push the metrics to the pushgateway and it will retain the information so it can be scraped later on.

Deploying a basic functional pushgateway is fairly simple:

    kubectl create -f prometheus-monitoring-guide/pushgateway/

Forward the service port of your pushgateway pod:

    kubectl port-forward pushgateway-deployment-66478477fb-gmsld 9091:9091

And try to post a metric using `curl`, which is basically what your batch jobs will do:

    echo "some_metric 3.14" | curl --data-binary @- http://localhost:9091/metrics/job/some_job

Now, you can scrape this metric normally, even if the original source is no longer running:

    curl localhost:9091/metrics | grep some_metric
    # TYPE some_metric untyped
    some_metric{instance="",job="some_job"} 3.14

You just need to add the pushgateway as a regular scrape target in your Prometheus configuration to recover these additional metrics.

## Prometheus Persistent metrics storage {#prometheuspersistentmetricsstorage}

The Prometheus server will store the metrics in a local folder, for a period of 15 days, by default. Also keep in mind that the default local Pod storage is ephemeral, which means that if the pod is replaced for any reason, then all the metrics will be gone.

Any production-ready deployment requires you to configure a persistent storage interface that will be able to maintain historical metrics data and survive pod restarts.

Again, Kubernetes already provides an abstraction that will fulfill this role: the <a href="https://kubernetes.io/docs/concepts/storage/persistent-volumes/" target="_blank">PersistentVolume</a>

However, as we just discussed, the PersistentVolume is just an abstraction for a data volume, so you need a hardware/software stack that provides the actual volumes.

There are several ways to accomplish this:

*   Major cloud providers usually expose a volume orchestrator out of the box, so you don't need to install any additional software.
*   If you are using your own hosts and don't have a storage provider yet, there are open source and CNCF-approved solutions like <a href="https://github.com/rook/rook" target="_blank">Rook</a>
    *   You can also use <a href="https://rook.github.io/docs/rook/master/helm-operator.html" target="_blank">Helm to install Rook</a> if you want to get up and running quickly
*   Several commercial storage solutions like NetApp also offer a compatibility layer for Kubernetes persistent volumes.

If you are planning to use stateful storage, then you could use <a href="https://kubernetes.io/docs/tutorials/stateful-application/basic-stateful-set/" target="_blank">Kubernetes StatefulSets</a> rather than deployments. Every pod will be unambiguously tied to a separate PersistentVolume, so you can kill and recreate the pods and they will automatically attach the correct volume.

You can try to destroy the deployment and create a similar service using StatefulSets:

    kubectl delete -f prometheus-monitoring-guide/alertmanager-example/prometheus-example.yaml
    kubectl create -f prometheus-monitoring-guide/storage/prometheus-example.yaml

Every pod will create its own PersistentVolume:

    kubectl get pv
    NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM                                                   STORAGECLASS   REASON    AGE
    pvc-b5a4aaa1-9584-11e8-97c9-42010a800278   50Gi       RWO            Delete           Bound     default/prometheus-metrics-db-prometheus-deployment-0   standard                 4h
    pvc-f4ee61e4-9584-11e8-97c9-42010a800278   50Gi       RWO            Delete           Bound     default/prometheus-metrics-db-prometheus-deployment-1   standard                 4h

There are three key differences between the deployment we were using before and this stateful set:

The API object type:

    kind: StatefulSet

The VolumeClaim defining the storage that should be created for each pod in the set:

     volumeClaimTemplates:
      - metadata:
          name: prometheus-metrics-db
        spec:
          accessModes:
          - ReadWriteOnce
          resources:
            requests:
              storage: 50Gi

The Prometheus server parameters defining data directory and retention period:

          containers:
          - args:
            - --storage.tsdb.path=/data
            - --storage.tsdb.retention=400d

On average, Prometheus uses only around 1-2 bytes per sample. Thus, to <a href="https://prometheus.io/docs/prometheus/latest/storage/" target="_blank">plan the capacity of a Prometheus server</a>, you can use the rough formula:

    needed_disk_space = retention_time_seconds * ingested_samples_per_second * bytes_per_sample

Prometheus server(s) can also regularly forward the metrics to a remote endpoint and only store the last uncommited chunk of readings locally. Some cloud-scale / multisite Prometheus solutions like Cortex or Thanos solutions make use of this feature, we will cover them on the last chapter of this guide.

## Conclusions {#conclusions}

Part 1 of this guide explained the basics of the Prometheus service, its integration with the Kubernetes orchestrator and different microservices you can find on a typical cloud-scale deployment. With this discussion around adding alerts, dashboards and long term storage on top of the core Prometheus server here in Part 2, we are getting much closer to an viable monitoring solution.

In the next chapter we will explain how to use the Kubernetes version of the prometheus-operator, which enables faster deployment of a complete stack in a more automated, scalable and declarative way.

 [1]: #alertmanagerprometheusalertingforkubernetes
 [2]: #grafanakubernetesmonitoringprometheusdashboards
 [3]: #prometheusmetricsforephemeraljobspushgateway
 [4]: #prometheuspersistentmetricsstorage
 [5]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/kubernetes_prom_diagram2.png
 [6]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/kubernetes_prom_alert1.png
 [7]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/kubernetes_prom_alert2.png
 [8]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/kubernetes_prom_webhook.png
 [9]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/kubernetes_prom_grafana.png
 [10]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/kubernetes_prom_grafana2.png