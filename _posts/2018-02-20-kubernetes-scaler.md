---
ID: 6272
post_title: >
  How to build a Kubernetes Horizontal Pod
  Autoscaler using custom metrics
author: Mateo Burillo
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/kubernetes-scaler/
published: true
post_date: 2018-02-20 09:22:55
---
The default Kubernetes Horizontal Pod Scaler (HPA) uses CPU load, in this article we will show how to configure it to pivot over any other monitoring metric implementing and extending the Kubernetes custom metrics API. We will use the number of HTTP requests as target metric to scale pod deployment.

Do not confuse a Kubernetes scaler (this article) with a Kubernetes scheduler:

*   **Kubernetes Horizontal Pod Autoscaler or HPA:** Updates the number of pods (scale up / scale down) in response to a metric & threshold value.
*   **Kubernetes Scheduler:** Assigns newly created pods to Kubernetes nodes. You can also use <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-scheduler/" target="_blank">custom metrics to configure your Kubernetes scheduler</a>.

## Kubernetes pod scaling {#kubernetespodscaling}

Kubernetes has several mechanisms to control a group of identical pod instances (ReplicationControllers, ReplicaSets, Deployments). You can manually resize these groups at will, and of course, you can also configure a control entity that automatically increases or decreases pod count based on current demand.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/autoscaler_kubernetes.jpg" alt="Scaler Kubernetes HPA" width="430" height="400" class="alignleft size-full wp-image-6283" />][1]   
  
The <a href="https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/" target="_blank">Horizontal Pod Autoscaler</a> periodically queries the configured metric(s) and compares against the desired target value. You can also configure minimum and maximum number of pods, scaling cooldown/delay periods (to avoid sudden bursts of containers rapidly spawning and terminating, also known as thrashing), sync period, etc.

You can implicitly create an HPA just using a kubectl one-liner, like this one:

    kubectl autoscale rc foo --min=2 --max=5 --cpu-percent=80

[tweet_box design="default" float="none"]How to build a #Kubernetes Horizontal Pod Autoscaler (HPA) using custom metrics, #golang API server[/tweet_box] 

## Kubernetes autoscaling using custom metrics {#kubernetesautoscalingusingcustommetrics}

Kubernetes HPA is an extremely useful mechanism, but pivoting just over CPU metrics may not be that interesting for your specific use case.

As you might expect, these Kubernetes entities can be extended to support any metric or group of metrics you desire. For this tutorial we have decided to optimize over the number of HTTP requests per second. This way, if my web service experiences a sudden traffic burst, Kubernetes will automatically increase the number of servicing pods, improving my service quality, when the rush is over it will downsize again, reducing the operative costs on my side.

### Former custom metrics method vs Kubernetes 1.6+ {#formercustommetricsmethodvskubernetes16}

Please note that in Kubernetes versions previous to 1.6 custom metrics were implemented using Heapster Model API, this method <a href="https://github.com/kubernetes/heapster/blob/master/docs/model.md" target="_blank">is now deprecated</a>.

Here we will use a newer method that requires deploying an extended API server implementing the Kubernetes <a href="https://github.com/kubernetes-incubator/custom-metrics-apiserver" target="_blank">custom metric apiserver spec</a>.

### Kubernetes cluster requirements & testbed node {#kubernetesclusterrequirementstestbednode}

To implement custom metrics using this method your Kubernetes cluster needs to meet these requirements:

*   Kubernetes version 1.6 or higher
*   API aggregation layer enabled
*   The kube-controller-manager needs the following flags:

*   *--horizontal-pod-autoscaler-use-rest-clients* should be true
*   *--kubeconfig "path-to-kubeconfig"* OR *--master "ip-address-of-apiserver"*

Do you have a compatible cluster? excellent! continue reading.

What if you don't have a compatible cluster yet? No problem, go to the [appendix][2] and configure a single-node compatible Kubernetes cluster using a virtual machine.

<a name="autoscaler"></a> 
## Deploying a custom Kubernetes autoscaler {#Deploying-a-custom-Kubernetes-autoscaler}

### Kubernetes scaler target deployment {#kubernetesscalertargetdeployment}

First you need a service / deployment that will be controlled by the HPA. In case you are using an empty testbed node, you can use the simple flask web server example included in the github repository:

    git clone https://github.com/mateobur/kubernetes-scaler-metrics-api.git
    $ kubectl create -f kubernetes-scaler-metrics-api/scaler/flask.yaml
    
    $ kubectl get pods
    NAME                     READY     STATUS    RESTARTS   AGE
    flask-84fccfd9bb-5xhx8   1/1       Running   0          3m
    flask-84fccfd9bb-dnxk6   1/1       Running   0          3m
    
    $ kubectl get svc
    NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
    flask        ClusterIP   10.101.236.79   <none>        5000/TCP   4m
    kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP    54m
    
    $ curl 10.101.236.79:5000
    Docker Flask Hello World

### Custom metrics API server {#custommetricsapiserver}

Next step is to create an entity that **extends the API** implementing the custom-metrics-apiserver Golang interface.

There is a <a href="https://github.com/directXMan12/k8s-prometheus-adapter" target="_blank">Prometheus adapter implementation</a> that will query local Prometheus servers. For this article we want to demonstrate a different implementation of the API server that directly retrieves metrics from Sysdig Monitor if you are using our Kubernetes monitoring product. This way you don't need to instrument the target pods in any way, as Sysdig provides HTTP request count and many other metrics automatically.

Naturally, you first need to install the Sysdig agent in your Kubernetes cluster to start collecting metrics. Make sure the pods and metrics you want to target show up in your Sysdig Monitor web panel before proceeding.

This will create a separate namespace for the pod implementing the API, and the required RBAC roles, permissions, bindings, etc:

    $ kubectl create -f kubernetes-scaler-metrics-api/scaler/custom-metrics-ns-rbac.yaml

And now, the key component, a pod extending and implementing the Kubernetes custom metrics API.

First let's configure the target metric, deployment and other parameters. Edit the file *kubernetes-scaler-metrics-api/scaler/custom-metric-api-sysdig.yaml*, you will find a Kubernetes secret that will contain your Sysdig API access token, please note that you need the base64 representation of this token:

    $ echo ffffffff-ffff-ffff-ffff-ffffffffffff | base64
    ZmZmZmZmZmYtZmZmZi1mZmZmLWZmZmYtZmZmZmZmZmZmZmZmCg==

Use your unique key in the command above and replace the placeholder text (*base64_version_of_your_sysdig_API_key_here*) with the resulting string, make sure there are no trailing spaces at the end.

Around line 30 of this file you will find:

<script src="https://gist.github.com/58d9d5cbd0ac2b20d6b34555e4a34476.js"></script> <noscript>
  <pre><code>
File: custommetrics.yml
-----------------------

- name: custom-metrics-server
  image: mateobur/custom-metrics-adapter:latest
  args:
  - --deploymentname=flask
  - --servicename=flask
  - --kubernetesnamespace=default
  - --targetmetric=net.http.request.count
  - --metrics-relist-interval=30s
  - --v=10
  - --logtostderr=true
</code></pre>
</noscript>

Most parameters, are self-explanatory. You can configure the Kubernetes deployment name, service name and namespace of the target deployment to scale. The metric you want to target (yes, **any metric** from Sysdig Monitor), refresh interval and log level.

Adjust those parameters to your liking. The default flask example should work just filling the API key secret.

Now, let's deploy the API server:

    $ kubectl create -f kubernetes-scaler-metrics-api/scaler/custom-metric-api-sysdig.yaml -n custom-metrics

Once the pod is *Running*, you should be able to inspect the metrics present in the *custom.metrics.k8s.io* extension:

    $ curl -sSk https://10.96.0.1/apis/custom.metrics.k8s.io/v1beta1
    {
      "kind": "APIResourceList",
      "apiVersion": "v1",
      "groupVersion": "custom.metrics.k8s.io/v1beta1",
      "resources": [
        {
          "name": "services/net.http.request.count",
          "singularName": "",
          "namespaced": true,
          "kind": "MetricValueList",
          "verbs": [
            "get"
          ]
        },
        {
          "name": "deployments.extensions/net.http.request.count",
          "singularName": "",
          "namespaced": true,
          "kind": "MetricValueList",
          "verbs": [
            "get"
          ]
        },
        {
          "name": "namespaces/net.http.request.count",
          "singularName": "",
          "namespaced": false,
          "kind": "MetricValueList",
          "verbs": [
            "get"
          ]
        }
      ]
    }

You can even increase the privileges of the default system user (don't do this in production) and fetch the specific metric value:

    $ kubectl create clusterrolebinding cli-api-read-access --clusterrole custom-metrics-getter --user system:anonymous
    $ curl -sSk https://10.96.0.1/apis/custom.metrics.k8s.io/v1beta1/namespaces/default/services/flask/net.http.request.count
    {
      "kind": "MetricValueList",
      "apiVersion": "custom.metrics.k8s.io/v1beta1",
      "metadata": {
        "selfLink": "/apis/custom.metrics.k8s.io/v1beta1/namespaces/default/services/flask/net.http.request.count"
      },
      "items": [
        {
          "describedObject": {
            "kind": "Service",
            "namespace": "default",
            "name": "flask",
            "apiVersion": "/__internal"
          },
          "metricName": "net.http.request.count",
          "timestamp": "2018-02-13T10:45:26Z",
          "value": "55"
        }
      ]
    }

Each pod is receiving an average of 55 HTTP requests for the interval we configured. Don't worry if your value is *0*, I just generated some random traffic for the example.

### Kubernetes Horizontal Pod Autoscaler: the scaler {#kuberneteshorizontalpodautoscalerthescaler}

We are only missing the last piece, the *Horizontal Pod Autoscaler* itself.

    kind: HorizontalPodAutoscaler
    apiVersion: autoscaling/v2beta1
    metadata:
      name: flask-hpa
    spec:
      scaleTargetRef:
        kind: Deployment
        name: flask
      minReplicas: 2
      maxReplicas: 10
      metrics:
      - type: Object
        object:
          target:
            kind: Service
            name: flask
          metricName: net.http.request.count
          targetValue: 100

It targets the *net.http.request.count* metric with a target value of 100 requests per minute, if the averaged value of the service gets higher, it will automatically spawn more pods, if enough pods are idle, it will kill some replicas.

    $ kubectl create -f kubernetes-scaler-metrics-api/scaler/horizontalpodautoscaler.yaml
    Wait a couple of minutes and can execute kubectl describe hpa to get current status.
    $ kubectl describe hpa
    Name:                                         flask-hpa
    Namespace:                                    default
    Labels:                                       <none>
    Annotations:                                  <none>
    CreationTimestamp:                            Tue, 13 Feb 2018 12:02:13 +0100
    Reference:                                    Deployment/flask-deploy
    Metrics:                                      ( current / target )
      "net.http.request.count" on Service/flask:  1 / 100
    Min replicas:                                 2
    Max replicas:                                 10
    Conditions:
      Type            Status  Reason            Message
      ----            ------  ------            -------
      AbleToScale     True    SucceededRescale  the HPA controller was able to update the target scale to 2
      ScalingActive   True    ValidMetricFound  the HPA was able to succesfully calculate a replica count from Service metric net.http.request.count
      ScalingLimited  True    TooFewReplicas    the desired replica count was less than the minimum replica count
    Events:
      Type    Reason             Age   From                       Message
      ----    ------             ----  ----                       -------
      Normal  SuccessfulRescale  5s    horizontal-pod-autoscaler  New size: 2; reason: All metrics below target

So, it seems that the HPA found the custom metric and is able to pivot over it, hurrah! You can create some network stress using, for example, Apache Benchmark (your specific service IP will vary):

    apt-get install apache2-utils
    ab -c 10 -t 300 -n 10000000 http://10.101.236.79:5000/

Leave the script running and open a new terminal. The number of requests will skyrocket, after a while you can query the scaler again and you will read:

    Type     Reason                        Age              From                       Message
    ----     ------                        ----             ----                       -------
    Normal   SuccessfulRescale             4m               horizontal-pod-autoscaler  New size: 3; reason: Service metric net.http.request.count above target
    Normal   SuccessfulRescale             27s              horizontal-pod-autoscaler  New size: 7; reason: Service metric net.http.request.count above target

It's scaling up the number of replicas, note that there is a cooldown period before the scaler takes a new decision, regardless of metrics. Eventually, if you wait 10-15 minutes after the stress script has finished, the scaler will realize it doesn't need the extra pods anymore:

    Normal   SuccessfulRescale             6m               horizontal-pod-autoscaler  New size: 5; reason: All metrics below target
    Normal   SuccessfulRescale             1m               horizontal-pod-autoscaler  New size: 2; reason: All metrics below target

### Sysdig Monitor dashboards & alerts {#sysdigmonitordashboardsalerts}

From the Sysdig Monitor side you can easily retrieve detailed information for this scaling group and related events.

For example, this would be an example dashboard that you can compile in a few minutes using native Kubernetes context for the filters and segments:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/kubernetes_scaler_sysdig_low.png" alt="Kubernetes Scaler Sysdig Low traffic" width="2173" height="1114" class="alignleft size-full wp-image-6287" />][3]   
  
Let's see what happens at the traffic peak:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/kubernetes_scaler_sysdig_high.png" alt="Kubernetes scaler sysdig high traffic" width="2180" height="1109" class="alignleft size-full wp-image-6289" />][4]   
  
And it's not just informative panels and diagrams, Sysdig Monitor understands Kubernetes events out of the box, like a *CrashloopBackoff* or the scaling events we are causing. For example, you can easily set an alert when the Kubernetes HPA gets busier than usual:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/kubernetes_scaler_sysdig_events.png" alt="Kubernetes scaler sysdig events" width="2300" height="1123" class="alignleft size-full wp-image-6290" />][5]   
  
Now you can configure a Kubernetes scaler to optimize over any of your application specific metrics!

### Kubernetes custom metrics API implementation {#kubernetescustommetricsapiimplementation}

Without going into full detail, the basic concept is that you need to implement <a href="https://github.com/kubernetes-incubator/custom-metrics-apiserver/blob/master/pkg/provider/interfaces.go" target="_blank">this metric provider interface</a>

<script src="https://gist.github.com/23703378e5f9ee1b01591a8b0351fd2d.js"></script> <noscript>
  <pre><code>
File: CustomMetricsProvider.go
------------------------------

type CustomMetricsProvider interface {
    // GetRootScopedMetricByName fetches a particular metric for a particular root-scoped object.
    GetRootScopedMetricByName(groupResource schema.GroupResource, name string, metricName string) (*custom_metrics.MetricValue, error)

    // GetRootScopedMetricByName fetches a particular metric for a set of root-scoped objects
    // matching the given label selector.
    GetRootScopedMetricBySelector(groupResource schema.GroupResource, selector labels.Selector, metricName string) (*custom_metrics.MetricValueList, error)

    // GetNamespacedMetricByName fetches a particular metric for a particular namespaced object.
    GetNamespacedMetricByName(groupResource schema.GroupResource, namespace string, name string, metricName string) (*custom_metrics.MetricValue, error)

    // GetNamespacedMetricByName fetches a particular metric for a set of namespaced objects
    // matching the given label selector.
    GetNamespacedMetricBySelector(groupResource schema.GroupResource, namespace string, selector labels.Selector, metricName string) (*custom_metrics.MetricValueList, error)

    // ListAllMetrics provides a list of all available metrics at
    // the current time.  Note that this is not allowed to return
    // an error, so it is reccomended that implementors cache and
    // periodically update this list, instead of querying every time.
    ListAllMetrics() []MetricInfo
}
</code></pre>
</noscript>

… to complete the <a href="https://github.com/kubernetes-incubator/custom-metrics-apiserver" target="_blank">boilerplate API server cod</a>e provided by the Kubernetes project.

For this example, we call the <a href="https://sysdig.gitbooks.io/sysdig-cloud-api/content/rest_api.html" target="_blank">Sysdig REST API</a> with the user-defined parameters configured in the YAML file (see above) and use this values to fill the MetricValue data structure expected by the code:

<script src="https://gist.github.com/6dae6485464d4502df4c9f0f7e18ed39.js"></script> <noscript>
  <pre><code>
File: MetricValue.go
--------------------

&custom_metrics.MetricValue{
            DescribedObject: custom_metrics.ObjectReference{
                APIVersion: info.GroupResource.Group + "/" + runtime.APIVersionInternal,
                Kind:       kind.Kind,
                Name:       name,
                Namespace:  namespace,
            },
            MetricName: info.Metric,
            Timestamp:  metav1.Time{time.Now()},
            Value:      *resource.NewMilliQuantity(int64(metricdatapoint*1000.0), resource.DecimalSI),
        }
</code></pre>
</noscript>

The complete component diagram looks like this:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/Kubernetes_scaler_diagram_sysdig.png" alt="Kubernetes scaler diagram Sysdig" width="1273" height="839" class="alignleft size-full wp-image-6292" />][6]   
  
## Further thoughts {#furtherthoughts}

*   The new "custom metrics API" method is more complex to deploy (at least the first), but provides a higher level of abstraction and flexibility.

*   The server code is a PoC, if you want to try this in a real environment you can use it as a starting point, but it doesn't have any error management or fallback code. We plan to keep improving this code as gets broader adoption but please, don’t hesitate to send over your pull requests or comments to us! Either via <a href="https://slack.sysdigrp2rs.wpengine.com/" target="_blank">Slack</a> or Twitter on <a href="https://twitter.com/sysdig" target="_blank">@sysdig</a>.

*   Being able to optimize over ANY custom metric(s) feels so effective, you are probably thinking about your particular use cases now.

<a name="appendix"></a> 
## Appendix: Kubernetes scaler testbed node {#appendix}

Ok, so you don't have a compatible cluster just yet. Let's create one.

First thing you need is a virtual machine, we are going to use Ubuntu server LTS 64 bits. If possible, allocate 2 virtual CPUs and ~3GB of RAM, your cluster may have pod allocation problems with less than that.

Do not configure any swap memory, Kubernetes doesn't like swap. If the host you want yo use already has swap space, you can always:

    swapoff -a

If you want to use *kubectl* outside the virtual machine, make sure that *hostname -i* resolves to the IP you want to target.

Once the VM is installed, download *git* and get the tutorial repository:

    sudo apt-get update
    sudo apt-get install git
    git clone https://github.com/mateobur/kubernetes-scaler-metrics-api.git
    

Next, you install Docker and kubeadm (as root), the custom-metrics specification is still in beta so we pin specific Kubernetes package versions to avoid inconsistencies. 

    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
    cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
    deb http://apt.kubernetes.io/ kubernetes-xenial main
    EOF
    apt-get update
    apt-get install -y docker.io
    apt-get install kubeadm=1.8.7-00 kubernetes-cni=0.5.1-00 kubelet=1.8.7-00 kubectl=1.8.7-00

and init your Kubernetes cluster!

    # kubeadm init --config kubernetes-scaler-metrics-api/testbednode/kubeadm.yml

This will take a while.

Look for the sentence:

    Your Kubernetes master has initialized successfully!

And pay attention to the text below it, it has instructions to configure *kubectl*, join nodes to the cluster, etc.

**Acknowledgments:** Some of the YAML files used in this article (Node configuration, RBAC roles) are based on the ones provided by Lucas on this <a href="https://github.com/luxas/kubeadm-workshop" target="_blank">tutorial</a>. Thanks Lucas!

change to the regular user and type:

    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

then try to inspect the *kube-system* pods, you should get something similar to this:

    $ kubectl get pods --all-namespaces
    NAMESPACE     NAME                               READY     STATUS    RESTARTS   AGE
    kube-system   etcd-kubenode                      1/1       Running   0          3m
    kube-system   kube-apiserver-kubenode            1/1       Running   0          3m
    kube-system   kube-controller-manager-kubenode   1/1       Running   0          4m
    kube-system   kube-dns-86cc76f8d-hmr2r           0/3       Pending   0          4m
    kube-system   kube-proxy-zctpb                   1/1       Running   0          4m
    kube-system   kube-scheduler-kubenode            1/1       Running   0          3m

Next, you need to deploy a pod network layer

    $ kubectl apply -f https://git.io/weave-kube-1.6

If you want to just use this single node, you need to remove the taint so regular pods can be scheduled in it:

    $ kubectl taint nodes --all node-role.kubernetes.io/master-

wait until all the pods are in *READY* status (the *kube-dns* ones usually take a little longer).

Your cluster is ready to go! make a snapshot just in case and [go back to configuring your custom metrics][7].

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/autoscaler_kubernetes.jpg
 [2]: #appendix
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/kubernetes_scaler_sysdig_low.png
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/kubernetes_scaler_sysdig_high.png
 [5]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/kubernetes_scaler_sysdig_events.png
 [6]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/02/Kubernetes_scaler_diagram_sysdig.png
 [7]: #autoscaler