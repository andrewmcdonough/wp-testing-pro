---
ID: 5503
post_title: >
  Integrating Prometheus alerts and events
  with Sysdig Monitor
author: Mateo Burillo
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/integrating-prometheus-alerts/
published: true
post_date: 2017-12-13 02:51:01
---
## Prometheus alerts: Sysdig ♥ Prometheus (part II) {#prometheusalertssysdigprometheuspartii}

If you already use (or plan to use) <a href="https://prometheus.io" target="_blank">Prometheus</a> alerts and events for application performance monitoring in your Docker / Kubernetes containers, you can easily integrate them with Sysdig Monitor via the Alertmanager daemon, we will showcase the integration in this post.

You can consider this piece the second part to the "<a href="https://sysdigrp2rs.wpengine.com/blog/prometheus-metrics/" target="_blank">Prometheus metrics: instrumenting your app with custom metrics and autodiscovery on Docker containers</a>" blog post, where we detailed how to integrate custom Prometheus metrics.

## Prometheus alert integration for your Docker and Kubernetes monitoring needs {#prometheusalertintegrationforyourdockerandkubernetesmonitoringneeds}

Prometheus provides its own alerting system using a separate daemon called <a href="https://prometheus.io/docs/alerting/alertmanager/" target="_blank">Alertmanager</a>. What happens if you already have a Prometheus monitoring infrastructure for APM and plan to integrate the <a href="https://sysdigrp2rs.wpengine.com/" target="_blank">Sysdig container intelligence platform</a>? (Or the other way around).

**They can work together without any migration or complex adaptation efforts**, actually, there is a lot to be gained from the combination of application-specific custom Prometheus monitoring that your developers love and deep container and service visibility provided by Sysdig.

These two contexts together add more dimensions to your monitoring data. To illustrate what we mean: You can easily detect that the MapReduce function on your backend container is taking longer than usual because your `kubernetes.replicas.running < kubernetes.replicas.desired`, the horizontal container scaling is failing and thus, the container that fired the alarm is receiving an order of magnitude more work.

[tweet_box design="default" float="none"]#Winning Manage infrastructure and application performance monitoring alerting by integrating @PrometheusIO alerts with Sysdig Monitoring[/tweet_box] 

## Metrics Exporters, Prometheus, Alertmanager & Sysdig integration {#metricsexportersprometheusalertmanagersysdigintegration}

### Docker monitoring scenario {#dockermonitoringscenario}

To put things in context let's assume that you already have a Docker environment instrumented with Prometheus:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/prometheus_alerts.png" alt="Prometheus alerts diagram" width="1157" height="377" class="alignleft size-full wp-image-5511" />][1] 
It could be Swarm, Kubernetes, Amazon ECS… whatever you're using our integration with prometheus works the same way.

Simplifying, you have several exporters that emit metrics, the Prometheus server aggregates them and checks the <a href="https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/" target="_blank">alert conditions</a>, if those conditions are met, it sends the configured alert to Alertmanager. Alertmanager is in charge of alert filtering, silencing, cooldown times and also sending the alert notifications to its **receivers**, mail and slack chat in our example.

One of the available receivers for Alertmanager is a <a href="https://en.wikipedia.org/wiki/Webhook" target="_blank">webhook</a>, this method boils down to HTTP POSTing a JSON data structure. Its simplicity and standard format provide a lot of flexibility to integrate any pair of producer / consumer software.

Accordingly, this is what we want to deploy:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/prometheus_alerts_to_sysdig.png" alt="Prometheus alert with Sysdig integration" width="721" height="288" class="alignleft size-full wp-image-5510" />][2]   
  
A new webhook AlertManager receiver that retrieves the JSON, reformats it to adapt to the <a href="https://support.sysdigrp2rs.wpengine.com/hc/en-us/sections/201140413-API-Guide" target="_blank">Sysdig API</a> function and uploads the alert data to Sysdig Monitor, is really just that. The interesting bit is that you don't have to modify the monitoring / alerting infrastructure you already have. It's just a new data output.

Let's create an easily reproducible, please-do-try-this-at-home scenario:

### Prometheus Metric Exporter {#prometheusmetricexporter}

First, we need some data to be scrapped. You can reuse the <a href="https://sysdigrp2rs.wpengine.com/blog/prometheus-metrics/#how-to-add-custom-metrics-to-your-application" target="_blank">trivial python script</a> from the last article to get some available metrics.

You can get it as a docker container:

    docker run -d -p 9100:9100 mateobur/pythonmetric

If you try:

    curl localhost:9100

You should see some raw metrics:

    ...
    function_exec_time_count{func_name="func1"} 53.0
    function_exec_time_sum{func_name="func1"} 10.620916843414307
    function_exec_time_bucket{func_name="func2",le="0.005"} 0.0
    function_exec_time_bucket{func_name="func2",le="0.01"} 0.0
    function_exec_time_bucket{func_name="func2",le="0.025"} 0.0
    ...

### Prometheus server & alerts {#prometheusserveralerts}

Next, we are going to configure the Prometheus server itself. Take a look to the <a href="https://prometheus.io/docs/alerting/overview/" target="_blank">Prometheus alerting guide</a> if you want to go further than this example.

We are going to modify three sections of the main configuration file.

First, we declare the metrics endpoint we just mentioned:

      - job_name: 'pythonfunc'
    
        # metrics_path defaults to '/metrics'
        # scheme defaults to 'http'.
    
        static_configs:
          - targets: ['pythonmetrics:9100']

So prometheus will regularly scrape it.

Second, we declare a rule configuration file to load:

    rule_files:
       - "/etc/prometheus/alerts.yml"

And third, we list the Alertmanager where we want to deliver our alerts:

    alerting:
      alertmanagers:
      - static_configs:
        - targets:
           - alertmanager:9093

This configuration requires that the Prometheus container is able to resolve the *pythonmetrics* and *alertmanager* hostnames, don't worry much about it, using the <a href="https://docs.docker.com/compose/" target="_blank">docker-compose</a> file we provide below, everything should work out of the box.

This is the *alerts.yml* we are going to use for the example:

    groups:
    - name: example
      rules:
      - alert: Function exec time too long
        expr: function_exec_time_sum{job="pythonfunc"} > 0.5
        for: 1m
        annotations:
          severity: 5
          container_name: nginx-front
          host_mac: 08:00:27:b2:06:e8
          container_id: 8c72934ff648
          host_hostName: dockernode
          description: Function exec time too long
        labels:
          source: docker

Nothing too surprising here, it has a name, a description, a condition to fire expressed in the internal Prometheus language and a time bucket to evaluate.

We are going to use the *annotations* as *scope* for our Sysdig alert, our webhook script will also translate the '_' to '.' characters (they are not valid as annotation name), so we can have the exact same names than native Sysdig alerts.

### Alertmanager & webhook receivers - Prometheus alerts integration {#alertmanagerwebhookreceiversprometheusalertsintegration}

The third piece of this puzzle is the Alertmanager, you can read about its configuration <a href="https://prometheus.io/docs/alerting/configuration/" target="_blank">here</a>. Particularly, you can use the different routes and receivers in the routing tree to filter and classify the alerts and, for example, deliver alerts from different parts of your infrastructure to separate <a href="https://sysdigrp2rs.wpengine.com/blog/introducing-sysdig-teams/" target="_blank">Sysdig teams</a>.

In this example we are just going to configure a default webhook receiver:

      # A default receiver
      receiver: SysdigMonitor

And define it here:

    - name: 'SysdigMonitor'
      webhook_configs:
        - url: 'http://sysdigwebhook:10000'

Any time the Alertmanager needs to notify about an alert it will send a HTTP POST to that URL endpoint.

### JSON Prometheus alerts to Sysdig API {#jsonprometheusalertstosysdigapi}

For the last piece, we just need to catch that JSON output, do some minor rearrangements of the data and call the Sysdig API.

These ~30 lines of Python are enough to have a functioning starting point:

<script src="https://gist.github.com/8915ed2b4494f25d32f990e492a03214.js"></script> <noscript>
  <pre><code>
File: webhook_receiver.py
-------------------------

from flask import Flask, request
from sdcclient import SdcClient
import json, os

app = Flask(__name__)
sdclient = SdcClient(os.environ[&#39;SYSDIG_API_KEY&#39;])


@app.route(&#39;/&#39;, methods=[&#39;POST&#39;])
def handle_alert():
    data = json.loads(request.data)
    alertname = data[&#39;groupLabels&#39;][&#39;alertname&#39;]
    description = data[&#39;commonAnnotations&#39;][&#39;description&#39;]
    severity = int(data[&#39;commonAnnotations&#39;][&#39;severity&#39;])
    scope = &#39;&#39;
    for key in data[&#39;commonAnnotations&#39;]:
        if key == "description" or key == "severity":
            continue
        newname = key.replace(&#39;_&#39;, &#39;.&#39;)
        scope += newname + &#39; = "&#39; + data[&#39;commonAnnotations&#39;][key] + &#39;" and &#39;
    tags = data[&#39;commonLabels&#39;]
    response = sdclient.post_event(alertname, description, severity,
                                   scope[:-4], tags)
    if not response[0]:
        print "Error: " + response[1]
    return "OK"


if __name__ == &#39;__main__&#39;:
    app.run(app.run(host=&#39;0.0.0.0&#39;, port=10000))
</code></pre>
</noscript>

### Complete integration with Docker-compose {#completeintegrationwithdockercompose}

To spawn all the pieces of this example at once in a more convenient way, you can just use this *docker-compose* <a href="https://gist.github.com/mateobur/07c2f1a800e8c4cca8503096f5bdb34f" target="_blank">file</a>.

Just fill out the `SYSDIG_API_KEY` variable with your <a href="https://app.sysdigcloud.com/#/settings/user" target="_blank">token string</a>, and spawn it

    docker-compose up -d

Let's take a look at every step:

Accessing *<a href="http://localhost:9100" target="_blank">http://localhost:9100</a>* you should see the raw metrics again, as we mentioned earlier.

Accessing *<a href="http://localhost:9090/" target="_blank">http://localhost:9090/</a>* you get the Prometheus interface

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/prometheus_interface.png" alt="Prometheus interface metric" width="2500" height="1215" class="alignleft size-full wp-image-5509" />][3]   
  
If you click on the *Alerts* tab:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/prometheus_fired_alerts.png" alt="Prometheus alert fired" width="336" height="120" class="alignleft size-full wp-image-5508" />][4]   
  
Your alert has fired, nice.

Next step, you have the Alertmanager interface at *<a href="http://localhost:9093" target="_blank">http://localhost:9093</a>*

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/alertmanager_alerts.png" alt="Alertmanager Prometheus alert" width="1606" height="1000" class="alignleft size-full wp-image-5507" />][5]   
  
It looks like the Alertmanager is taking care of our alerts, if you click on *Status*, you can see the current configuration file with the routing tree and receivers.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/alertmanager_config.png" alt="Alertmanager Sysdig webhook " width="300" height="100" class="alignleft size-full wp-image-5506" />][6]   
  
Let's take a look at the docker-compose logs:

    $ docker-compose logs
    Attaching to sysdigwebhook_1, promserver_1, alertmanager_1, pythonmetrics_1
    sysdigwebhook_1  |  * Running on http://0.0.0.0:10000/ (Press CTRL+C to quit)
    sysdigwebhook_1  | 172.19.0.3 - - [12/Dec/2017 15:09:09] "POST / HTTP/1.1" 200

Your *sysdigwebhook* container has received a HTTP POST from the Alertmanager.

And the last and most important part, if you open your Sysdig Monitor Panel:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/sysdig_with_prometheus_event.png" alt="Sysdig Monitor with Prometheus alert integrated" width="1752" height="813" class="alignleft size-full wp-image-5513" />][7]   
  
There it is!

Your custom event will full scope and tags, on top of any other Sysdig metric you need to correlate.

## Supercharge Debugging {#superchargedebugging}

By adding multi-dimensional scope to your metrics and dashboards you can supercharge your debugging capacity and find data correlations that are extremely arduous to discover manually.

Also, webhooks are incredibly useful to easily integrate microservices.

The webhook receiver code is just a PoC, you can use it as an starting point, but make sure to add exception handling and fallback routines if you plan to do anything more serious than a local test.

Prometheus and Alertmanager are opensource and you can also get a <a href="https://sysdigrp2rs.wpengine.com/product/monitor/" target="_blank">free trial of Sysdig Monitor right away</a>.

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/prometheus_alerts.png
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/prometheus_alerts_to_sysdig.png
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/prometheus_interface.png
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/prometheus_fired_alerts.png
 [5]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/alertmanager_alerts.png
 [6]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/alertmanager_config.png
 [7]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/sysdig_with_prometheus_event.png