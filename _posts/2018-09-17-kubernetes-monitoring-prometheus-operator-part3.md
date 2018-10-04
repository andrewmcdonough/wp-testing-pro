---
ID: 10300
post_title: 'Kubernetes monitoring with Prometheus &#8211; Prometheus operator tutorial (part 3).'
author: Mateo Burillo
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/kubernetes-monitoring-prometheus-operator-part3/
published: true
post_date: 2018-09-17 09:19:32
---
We covered how to install a complete 'Kubernetes monitoring with Prometheus' stack in the previous chapters of this guide. But using the Prometheus Operator framework and its Custom Resource Definitions has significant advantages over manually adding metric targets and service providers, which can become cumbersome for large deployments and doesn't fully utilize Kubernetes' orchestrator capabilities.

We are going to deploy a similar stack but more automated and flexible this time. 

Previously, we covered:

<a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-monitoring-prometheus/" target="_blank" rel="noopener">1 – Kubernetes Monitoring with Prometheus, basic concepts and initial deployment</a>

<a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-monitoring-with-prometheus-alertmanager-grafana-pushgateway-part-2/" target="_blank" rel="noopener">2 – Kubernetes Monitoring with Prometheus: Alertmanager, Grafana, PushGateway</a>

3 – Prometheus Operator Tutorial: a fully automated Kubernetes deployment for Prometheus, Alertmanager and Grafana. (this one):

*   [What is a Kubernetes Operator][1]
*   [The Prometheus Operator][2]
*   [Prometheus Operator - Quick install][3]
*   [Auto-configuring metric endpoints][4]
*   [Alert rules and the Alertmanager component][5]
*   [Visualization and Dashboards with Grafana][6]

4 – Prometheus performance considerations, high availability, external storage, dimensionality limits. [tweet_box design="default" float="none"]Deploy a #Kubernetes monitoring with #Prometheus stack in a scalable, automated and elegant fashion using the Prometheus #Operator.[/tweet_box] 

## What is a Kubernetes Operator? {#whatisakubernetesoperator}

Operators are Kubernetes-specific applications (pods) that configure, manage and optimize other Kubernetes deployments automatically. They are implemented as a <a href="https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#custom-controllers" target="_blank" rel="noopener">custom controller</a>.

A Kubernetes operator encapsulates the know-how of deploying and scaling an application and directly executes algorithm decisions communicating with the API.

A Kubernetes Operator might be able to:

*   Install and provide sane initial configuration and sizing for your deployment, according to the specs of your Kubernetes cluster.
*   Perform live reloading of deployments and pods to accommodate for any user-requested parameter modification (hot config reloading).
*   Automatically scale up or down according to performance metrics.
*   Perform backups, integrity checks or any other maintenance task.

Basically, anything that can be expressed as code by a human admin can be automated inside a Kubernetes Operator.

Kubernetes Operators make extensive use of Custom Resource Definitions (or <a href="https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#customresourcedefinitions" target="_blank" rel="noopener">CRDs</a>) to create context-specific entities and objects that will be accessed like any other Kubernetes API resource. For example, in the next sections, you will be able to interact with a 'Prometheus' Kubernetes API object which defines the initial configuration and scale of a Prometheus server deployment. Operators read, write and update CRDs to persist service configuration inside the cluster.

## Prometheus Operator {#prometheusoperator}

The <a href="https://github.com/coreos/prometheus-operator" target="_blank" rel="noopener">Prometheus Operator for Kubernetes</a> provides easy monitoring definitions for Kubernetes services and deployment and management of Prometheus instances.

We are going to use the Prometheus Operator to:

*   Perform the initial installation and configuration of the full Kubernetes-Prometheus stack
    *   Prometheus servers
    *   Alertmanager
    *   Grafana
    *   Host node_exporter
    *   kube-state-metrics
*   Define metric endpoint autoconfiguration using the `ServiceMonitor` entities
*   Customize and scale the services using the Operator CRDs and ConfigMaps, making our configuration fully portable and declarative

The Operator acts on the following <a href="https://github.com/coreos/prometheus-operator/blob/master/README.md#customresourcedefinitions" target="_blank" rel="noopener">custom resource definitions (CRDs)</a>:

*   **Prometheus**, which defines the desired Prometheus deployment. The Operator ensures at all times that a deployment matching the resource definition is running.
*   **ServiceMonitor**, which declaratively specifies how groups of services should be monitored. The Operator automatically generates Prometheus scrape configuration based on the definition.
*   **PrometheusRule**, which defines a desired Prometheus rule file, which can be loaded by a Prometheus instance containing Prometheus alerting and recording rules.
*   **Alertmanager**, which defines a desired Alertmanager deployment. The Operator ensures at all times that a deployment matching the resource definition is running.

The <a href="https://github.com/coreos/prometheus-operator/tree/master/contrib/kube-prometheus" target="_blank" rel="noopener">kube-prometheus</a> directory inside the Operator repository contains default services and configurations, so you get not only the Prometheus Operator itself, but a complete setup that you can start using and customizing from the get-go.

## The Prometheus Operator - Components architecture diagram {#theprometheusoperatorcomponentsarchitecturediagram}

The following is a components diagram of the Kubernetes monitoring system that we intend to build:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/prometheus_operator_diagram-1170x995.png" alt="Prometheus Operator Diagram" width="840" height="744" class="alignleft size-large wp-image-10309" />][7]

You can see that this diagram is similar to <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-monitoring-with-prometheus-alertmanager-grafana-pushgateway-part-2/#prometheusmonitoringstackarchitectureoverview" target="_blank" rel="noopener">the one in part 2, where we were installing Prometheus components manually</a>. There are, however, two important differences:

1.  Different Prometheus deployments will monitor different resources: 
    *   One group of Prometheus servers (1 to N, depending on your scale) is going to monitor the Kubernetes internal component and state.
    *   Another group of Prometheus server is going to monitor any other app deployed in your cluster.
2.  Instead of directly creating Deployments and Services, we are going to use the CRDs to feed the Operator with metrics sources. In the example above we have one Grafana service with two data sources, but we will show how to rearrange the component diagram in a declarative way thanks to the Prometheus Operator.

## Prometheus Operator - Quick install {#prometheusoperatorquickinstall}

The <a href="https://github.com/mateobur/prometheus-monitoring-guide" target="_blank" rel="noopener">prometheus-monitoring-guide</a> repository contains some of the base yaml files and example customizations we will use. First of all, clone the repositories if you haven't done it yet:

    git clone https://github.com/coreos/prometheus-operator.git
    git clone https://github.com/mateobur/prometheus-monitoring-guide.git

Make sure your local `kubectl` is pointing to a running kubernetes cluster. You probably want to use a testbed cluster that you can easily discard and recreate, so you can experiment with different configurations.

To install *kube-prometheus* using defaults you just need to:

    kubectl create -f prometheus-operator/contrib/kube-prometheus/manifests/

The default stack deploys a lot of components out of the box:

    kubectl get pods -n monitoring
    NAME                                  READY     STATUS    RESTARTS   AGE
    alertmanager-main-0                   2/2       Running   0          7m
    alertmanager-main-1                   2/2       Running   0          7m
    alertmanager-main-2                   2/2       Running   0          6m
    grafana-5568b65944-nwbct              1/1       Running   0          8m
    kube-state-metrics-76bdcb8ff9-77t52   4/4       Running   0          6m
    node-exporter-224sh                   2/2       Running   0          7m
    node-exporter-2s89j                   2/2       Running   0          7m
    node-exporter-x8w8b                   2/2       Running   0          7m
    prometheus-k8s-0                      3/3       Running   1          7m
    prometheus-k8s-1                      3/3       Running   1          6m
    prometheus-operator-cdccdb8db-vcvhx   1/1       Running   0          8m

You can see:

*   The *prometheus-operator* pod, the core of the stack, in charge of managing other deployments like Prometheus servers or Alertmanager servers
*   A *node-exporter* pod per physical host (3 in this example)
*   A *kube-state-metrics* exporter
*   The default Prometheus server deployment *prometheus-k8s* (replicas: 2)
*   The default Alertmanager deployment *alertmanager-main* (replicas: 3)
*   The Grafana deployment *grafana* (replicas: 1)

### Accessing the interfaces of the Prometheus Operator {#accessingtheinterfacesoftheprometheusoperator}

To quickly get a glimpse of the interfaces that you just deployed, you can use the port-forward capabilities of `kubectl`.

    kubectl port-forward grafana-5568b65944-nwbct -n monitoring 3000:3000

Now point your web browser to <a href="http://localhost:3000" target="_blank" rel="noopener">http://localhost:3000</a> , you will access the Grafana interface, which is already populated with some useful dashboards!

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/prometheus_operator_grafana1-1170x606.png" alt="Prometheus Operator Grafana1" width="640" height="331" class="size-large wp-image-10310 aligncenter" />][8]

But If you want to deploy a production Kubernetes monitoring you will need to expose these interfaces properly using an ingress controller and with proper security: HTTPS certificates and authentication.

### Interacting with the Prometheus Operator via CRD objects {#interactingwiththeprometheusoperatorviacrdobjects}

To modify this Prometheus stack deployment, instead of modifying each component Deployment or StatefulSet as you would expect in Kubernetes, you will directly customize the abstract definitions and let the operator handle the orchestration for you.

Let's start with a simple example: let's say 3 Alertmanager pods are too many for this scenario, and we just want one.

First, we will explore the `alertmanager` CRD:

    kubectl get alertmanager main -n monitoring -o yaml
    apiVersion: monitoring.coreos.com/v1
    kind: Alertmanager
    metadata:
      clusterName: ""
      creationTimestamp: 2018-08-28T09:15:25Z
      labels:
        alertmanager: main
      name: main
      namespace: monitoring
      resourceVersion: "1401"
      selfLink: /apis/monitoring.coreos.com/v1/namespaces/monitoring/alertmanagers/main
      uid: e5d335aa-aaa2-11e8-8cf2-42010a800113
    spec:
      baseImage: quay.io/prometheus/alertmanager
      nodeSelector:
        beta.kubernetes.io/os: linux
      replicas: 3
      serviceAccountName: alertmanager-main
      version: v0.15.2

From here you can modify the pod labels, serviceAccount, <a href="https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#nodeselector" target="_blank" rel="noopener">nodeSelector</a> and what we were looking for: the number of replicas.

Change the *replicas* parameter to '1', patching the API object (you can find the patched version in the <a href="https://github.com/mateobur/prometheus-monitoring-guide" target="_blank" rel="noopener">prometheus-monitoring-guide</a> repository):

    kubectl create -f prometheus-monitoring-guide/operator/alertmanager.yaml --dry-run -o yaml | kubectl apply -f -

If you list the pods in the `monitoring` namespace again, you will see just one Alertmanager pod and the resize event in the logs of the Prometheus Operator itself:

    kubectl logs prometheus-operator-cdccdb8db-vcvhx -n monitoring | tail -n 1
    level=info ts=2018-08-28T09:47:20.523662127Z caller=operator.go:402 component=alertmanageroperator msg="sync alertmanager" key=monitoring/main

## Prometheus Operator endpoint to scrape autoconfiguration {#prometheusoperatorendpointtoscrapeautoconfiguration}

Next step is to build upon this configuration to start monitoring any other services deployed in your cluster.

There are two custom resources involved in this process:

*   The Prometheus CRD
    *   Defines Prometheus server pod metadata
    *   Defines # of Prometheus server replicas
    *   Defines Alertmanager(s) endpoint to send triggered alert rules
    *   Defines labels and namespace filters for the ServiceMonitor CRDs that will be applied by this Prometheus server deployment
        *   The ServiceMonitor objects will provide the dynamic target endpoint configuration
*   The ServiceMonitor CRD
    *   Filters endpoints by namespace, labels, etc
    *   Defines the different scraping ports
    *   Defines all the additional scraping parameters like scraping interval, protocol to use, TLS credentials, re-labeling policies, etc.

The Prometheus object filters and selects N ServiceMonitor objects, which in turn, filter and select N Prometheus metrics endpoints.

If there is a new metrics endpoint that matches the ServiceMonitor criteria, this target will be automatically added to all the Prometheus servers that select that ServiceMonitor.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/prometheus_operator_servicemonitor.png" alt="Prometheus Operator ServiceMonitor" width="942" height="441" class="size-full wp-image-10308 aligncenter" />][9]

As you can see in the diagram above, the ServiceMonitor targets Kubernetes services, not the endpoints directly exposed by the pod(s).

We already have a Prometheus deployment monitoring all the Kubernetes internal metrics (kube-state-metrics, node-exporter, Kubernetes API, etc) but now we need a separate deployment to take care of any other application running on top of the cluster.

In order to do this new deployment, first let's take a look at this Prometheus CRD before applying it to the cluster (you can find this file in the repository as shown below):

    apiVersion: monitoring.coreos.com/v1
    kind: Prometheus
    metadata:
      labels:
        app: prometheus
        prometheus: service-prometheus
      name: service-prometheus
      namespace: monitoring
    spec:
      alerting:
        alertmanagers:
        - name: alertmanager-main
          namespace: monitoring
          port: web
      baseImage: quay.io/prometheus/prometheus
      logLevel: info
      paused: false
      replicas: 2
      retention: 2d
      routePrefix: /
      ruleSelector:
        matchLabels:
          prometheus: service-prometheus
          role: alert-rules
      serviceAccountName: prometheus-k8s
      serviceMonitorSelector:
        matchExpressions:
        - key: serviceapp
          operator: Exists

For the sake of brevity, we will reuse the *alertmanager* and *serviceAccount* configuration found in the other deployment.

In this Prometheus server configuration file we can find:

*   The number of Prometheus replicas using this config (2)
*   The ruleSelector that will dynamically configure alerting rules
*   The Alertmanager deployment (could be more than one pod for redundancy) that will receive the triggered alerts
*   The **ServiceMonitorSelector**, this is, the filter that will decide if a given *serviceMonitor* will be used to configure this Prometheus server.
    *   For this example we have decided that a *serviceMonitor* will be associated with this Prometheus deployment if it contains the label *serviceapp* in its metadata.

Once tuned to our needs, we can apply the new configuration directly from the repository:

    kubectl create -f prometheus-monitoring-guide/operator/service-prometheus.yaml

Here the Prometheus Operator will notice the new API object and create the desired deployment for you:

    prometheus-service-prometheus-0       3/3       Running   1          12m
    prometheus-service-prometheus-1       3/3       Running   1          12m

If you connect to the interface of any of these pods, you will notice that we don't have any metrics target yet. We need a service to scrape. If you have something installed already and want to use it, great!. Otherwise, we can quickly get an app running using <a href="https://helm.sh/" target="_blank" rel="noopener">Helm</a>.

<a href="https://coredns.io/" target="_blank" rel="noopener">CoreDNS</a> is a fast and flexible DNS server, an incubating-level project of the Cloud Native Computing Foundation. So if you have helm setup with your cluster, just run:

    kubectl create ns coredns
    helm install --name coredns --namespace=coredns stable/coredns

CoreDNS exposes Prometheus metrics out of the box (using port 9153):

    kubectl get svc -n coredns
    NAME              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                  AGE
    coredns-coredns   ClusterIP   10.11.253.94   <none>        53/UDP,53/TCP,9153/TCP   3m

Now, you can connect this new service with your Services Prometheus deployment using a ServiceMonitor (you can find this file in the repository as shown below):

    apiVersion: monitoring.coreos.com/v1
    kind: ServiceMonitor
    metadata:
      labels:
        serviceapp: coredns-servicemonitor
      name: coredns-servicemonitor
      namespace: monitoring
    spec:
      endpoints:
      - bearerTokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
        interval: 15s
        port: metrics
      namespaceSelector:
        matchNames:
        - coredns
      selector:
        matchLabels:
          release: coredns

Apply it to your cluster:

    kubectl create -f prometheus-monitoring-guide/operator/servicemonitor-coredns.yaml

After a few seconds, if you look at the scraping targets inside the Prometheus interface, you will see how the configuration has been automatically updated and you are receiving metrics from this target:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/prometheus_operator_targets1-1170x206.png" alt="Prometheus Operator scrape target" width="640" height="113" class="size-large wp-image-10314 aligncenter" />][10]

Now, if we increase the number of serving pods (and thus, metrics endpoints) for this CoreDNS deployment:

    helm upgrade coredns stable/coredns --set replicaCount=3

Every target is automatically detected by the ServiceMonitor and registered in your Prometheus configuration:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/prometheus_operator_targets2-1170x191.png" alt="Prometheus Operator scrape Targets" width="640" height="104" class="size-large wp-image-10313 aligncenter" />][11]

## Prometheus Operator - How to configure Alert Rules {#prometheusoperatorhowtoconfigurealertrules}

We can configure Kubernetes monitoring alerts in our Prometheus deployments using a concept that is very similar to the ServiceMonitor: the PrometheusRule CRD.

When we were defining our Prometheus deployment there was a configuration block to filter and match these objects:

      ruleSelector:
        matchLabels:
          prometheus: service-prometheus
          role: alert-rules

If you define an object containing the PromQL rules you desire and matching the desired metadata, they will be automatically added to the Prometheus servers' configuration (you can find this file in the repository as shown below).

    apiVersion: monitoring.coreos.com/v1
    kind: PrometheusRule
    metadata:
      labels:
        prometheus: service-prometheus
        role: alert-rules
      name: prometheus-service-rules
      namespace: monitoring
    spec:
      groups:
      - name: general.rules
        rules:
        - alert: TargetDown-serviceprom
          annotations:
            description: '{{ $value }}% of {{ $labels.job }} targets are down.'
            summary: Targets are down
          expr: 100 * (count(up == 0) BY (job) / count(up) BY (job)) > 10
          for: 10m
          labels:
            severity: warning
        - alert: DeadMansSwitch-serviceprom
          annotations:
            description: This is a DeadMansSwitch meant to ensure that the entire Alerting
              pipeline is functional.
            summary: Alerting DeadMansSwitch
          expr: vector(1)
          labels:
            severity: none

Apply from the repository:

    kubectl create -f prometheus-monitoring-guide/operator/prometheusrules.yaml

You will be able to inspect these alerts right away from the service-prometheus interface. Use a local port-forward if you don't want to configure an external ingress:

    kubectl port-forward prometheus-service-prometheus-0 -n monitoring 9090:9090

Access the *Alerts* tab in the Prometheus server interface:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/prometheus_operator_alertmanager3.png" alt="Prometheus Operator AlertManager1" width="539" height="313" class="size-full wp-image-10305 aligncenter" />][12]

*DeadManSwitch* is a common name for an alert that will always fire (it has a firing condition that always evaluates to *true*) and it's there to check that your alerting pipeline is working as expected. A few seconds after this condition is loaded you should see the alert name over a light red background (firing), as in the image above.

If you open the Alertmanager interface now (port 9093), you can see the alerts coming from this Prometheus deployment. Use a local port-forward if you don't want to configure an external ingress:

    kubectl port-forward alertmanager-main-0 -n monitoring 9093:9093

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/prometheus_operator_alertmanager4.png" alt="Prometheus Operator AlertManager 2" width="539" height="174" class="size-full wp-image-10304 aligncenter" />][13]

## Prometheus Operator - Defining Grafana Dashboards {#prometheusoperatordefininggrafanadashboards}

As of today, there is no custom resource definition for the Grafana component in the Prometheus Operator. In order to manage Grafana configuration we will be using then Kubernetes secrets and ConfigMaps, including new datasources and new dashboards.

When using Grafana deployed using the Prometheus Operator, datasources are defined as data structure encoded using base64 that Grafana reads from a Kubernetes secret.

If you decode your current secret data, you should see something similar to this:

    kubectl get secret grafana-datasources -n monitoring -o jsonpath --template '{.data.prometheus\.yaml}' | base64 -d
    {
        "apiVersion": 1,
        "datasources": [{
            "access": "proxy",
            "editable": false,
            "name": "prometheus",
            "orgId": 1,
            "type": "prometheus",
            "url": "http://prometheus-k8s.monitoring.svc:9090",
            "version": 1
        }]
    }

We just need to append the new datasource using the same JSON format, re-encode the secret data as base64 and update the secret.

Let's create this new file:

    {
        "apiVersion": 1,
        "datasources": [
            {
                "access": "proxy",
                "editable": false,
                "name": "prometheus",
                "orgId": 1,
                "type": "prometheus",
                "url": "http://prometheus-k8s.monitoring.svc:9090",
                "version": 1
            },
            {
                "access": "proxy",
                "editable": false,
                "name": "service-prometheus",
                "orgId": 1,
                "type": "prometheus",
                "url": "http://service-prometheus.monitoring.svc:9090",
                "version": 1
            }
        ]
    }

Encapsulate it in a secret object (you can use your own file or the one from the repository as shown below):

    kubectl create secret generic grafana-datasources -n monitoring --from-file=./prometheus-monitoring-guide/operator/grafana/prometheus.yaml --dry-run -o yaml > grafana-datasources.yaml

And patch the Grafana secret in the API:

    kubectl patch secret grafana-datasources -n monitoring --patch "$(cat grafana-datasources.yaml)"

Note that the datasource points to a service, not the Prometheus pods, you will have to create the `service-prometheus.monitoring.svc` service using this file:

    kubectl create -f prometheus-monitoring-guide/operator/prometheus-svc.yaml

Finally, restart Grafana:

    kubectl delete pod grafana-5568b65944-szhx4 -n monitoring

And go to the *datasources* tab in the UI (Use the port-forward again if necessary, port 3000), you should see both Prometheus deployments as data sources:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/prometheus_operator_grafana2.png" alt="Prometheus Operator Grafana Datasources" width="983" height="431" class="size-full wp-image-10303 aligncenter" />][14]

Using the Prometheus Operator, Grafana dashboard can be externally loaded just encoding the Dashboard JSON data inside a Kubernetes ConfigMap:

    kubectl get cm -n monitoring
    NAME                                        DATA      AGE
    grafana-dashboard-k8s-cluster-rsrc-use      1         20d
    grafana-dashboard-k8s-node-rsrc-use         1         20d
    grafana-dashboard-k8s-resources-cluster     1         20d
    grafana-dashboard-k8s-resources-namespace   1         20d
    grafana-dashboard-k8s-resources-pod         1         20d

These ConfigMaps are explicitly mounted in the Grafana deployment definition:

    kubectl get deployments grafana -n monitoring -o yaml
    ...
            - mountPath: /grafana-dashboard-definitions/0/pods
              name: grafana-dashboard-pods
    ...
          - configMap:
              defaultMode: 420
              name: grafana-dashboard-pods
            name: grafana-dashboard-pods

You can create a new ConfigMap per dashboard and add the corresponding entries in the deployment definition that manages the Grafana pods.

Another, maybe more flexible option, is to use a Grafana Helm chart that will take care of your <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-monitoring-with-prometheus-alertmanager-grafana-pushgateway-part-2/#grafanakubernetesmonitoringprometheusdashboards" target="_blank" rel="noopener">Dashboard configuration upgrades automatically, as we described in part 2 of this guide.</a>

## Conclusions {#conclusions}

Using the Prometheus Operator we have managed to build the Kubernetes monitoring stack with less effort, in a more declarative and reproducible way, which is also easier to scale, modify or migrate to a different set of hosts.

So far we have been talking about the features, advantages and compelling possibilities of monitoring a Kubernetes cluster using the Prometheus technology stack. Before committing to a large deployment though, you also need to be aware of its shortcomings or caveats, for example:

*   **Long term storage:** As we have mentioned before, Prometheus abstracts away the long term metrics storage (and closely related concerns like backup, data redundancy, trend analysis, data mining). Part 2 of this guide describes a minimal Rook deployment that will provide the basic building blocks of a complete storage solution.
*   **Authorization and Authentication:** As pointed out in the project documentation, Prometheus and its components do not provide any server-side authentication, authorization or encryption. No groups of users with different levels of access or any other RBAC framework.
*   **Vertical and Horizontal scalability:** Not so much of a drawback but rather an aspect that you need to plan in advance to make informed decisions about your target capacity. We will cover Prometheus performance considerations, high availability and federation in the last part of this guide.

For a more comprehensive comparison of Prometheus vs a commercial monitoring platform like Sysdig, please read: <a href="https://sysdigrp2rs.wpengine.com/blog/prometheus-monitoring-and-sysdig-monitor-a-technical-comparison/" target="_blank" rel="noopener">Prometheus Monitoring and Sysdig Monitor: A Technical Comparison</a>

 [1]: #whatisakubernetesoperator
 [2]: #prometheusoperator
 [3]: #prometheusoperatorquickinstall
 [4]: #prometheusoperatorendpointtoscrapeautoconfiguration
 [5]: #prometheusoperatorhowtoconfigurealertrules
 [6]: #prometheusoperatordefininggrafanadashboards
 [7]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/prometheus_operator_diagram.png
 [8]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/prometheus_operator_grafana1.png
 [9]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/prometheus_operator_servicemonitor.png
 [10]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/prometheus_operator_targets1.png
 [11]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/prometheus_operator_targets2.png
 [12]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/prometheus_operator_alertmanager3.png
 [13]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/prometheus_operator_alertmanager4.png
 [14]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/prometheus_operator_grafana2.png