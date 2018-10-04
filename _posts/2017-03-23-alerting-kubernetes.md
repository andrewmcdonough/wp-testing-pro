---
ID: 3912
post_title: 'Monitoring Kubernetes (part 2): Best practices for alerting on Kubernetes'
author: Jorge Salamero Sanz
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/alerting-kubernetes/
published: true
post_date: 2017-03-23 14:53:53
---
A step by step cookbook on how to configure alerting in your Kubernetes cluster with a focus on the infrastructure layer.

*This article is part of our series on operating Kubernetes in production. Part 1 covered the basics of <a href="/blog/monitoring-kubernetes-with-sysdig-cloud/" target="_blank">Kubernetes and monitoring tools</a>; this part covers Kubernetes alerting best practices. Next we’ll cover <a href="/blog/kubernetes-service-discovery-docker/" target="_blank">troubleshooting Kubernetes service discovery</a>, and finally the final section is <a href="/blog/monitoring-docker-kubernetes-wayblazer/" target="_blank">a real-world use case of monitoring Kubernetes</a>.*

Monitoring is a fundamental piece in every successful infrastructure. Monitoring is the base of hierarchy of reliability. Monitoring helps reducing response time to incidents, enabling detecting, troubleshooting and debugging systemic problems. In the Cloud times where infrastructure is highly dynamic, monitoring is also fundamental for capacity planning.

Effective alerting is at the bedrock of a monitoring strategy. Naturally, with the shift to containers and Kubernetes-orchestrated environments, your alerting strategy will need to evolve as well.

This is due to a few core reasons, many of which we covered in <a href="/blog/monitoring-kubernetes-with-sysdig-cloud/" target="_blank">How To Monitor Kubernetes</a>:

*   **Visibility:** Containers are black boxes. Traditional tools can only check against public monitoring endpoints. If you want to deeply monitor the service in question, you need to take a different approach.
*   **New infrastructure layers:** Between your services and the host now you have a new layer: the containers and the container orchestrator. These are new internal services that you need to monitor and your monitoring system needs to understand them.
*   **Dynamic rescheduling:** Container are not coupled with nodes like services where before, so traditional monitoring doesn’t work effectively. There is no a static endpoint where service metrics are available, no static number of service instances running (think of a canary deployment or auto-scaling setup). It is fine that a process is being killed in one node because most of the chances are that is being rescheduled somewhere else in your infrastructure.
*   **Metadata and labels:** With services spread across multiple containers, monitoring system level and service specific metrics for all of those, plus all the new services that Kubernetes brings in, how to give all these information some sense for our own sanity? Sometimes you want to see a metric like network requests across a service distributed in containers in different nodes, sometimes you want to see the same metric for all containers in a specific node, no matter what service they belong to. This is basically a multi-dimensional metric system. You need to look at the metrics from different perspectives. If we automatically tag metrics with the different labels existing in Kubernetes and our monitoring system understands Kubernetes metadata, we just came up with a way of aggregating and segmenting metrics as required on each situation.

With these issues in mind, let’s create a set of alerts that are essential to any Kubernetes environment. Our Kubernetes alerts tutorial will cover:

*   [Alerting on application layer metrics][1]
*   [Alerting on services running on Kubernetes][2]
*   [Alerting on the Kubernetes infrastructure][3]
*   [Alerting on the host / node layer][4]

As a bonus, we’ll also look into how your [alerting can accelerate troubleshooting by monitoring your syscalls][5] around problems. Let’s dig in! [tweet_box design="default" float="none"] 4 layers to monitor in #Kubernetes: application KPIs, services, orchestration and nodes [/tweet_box] 

## Alerting on application layer metrics {#apps}

Metrics that allow you to confirm that your application performing as expected are known as working metrics. These metrics typically come from your users’ or consumers’ expected actions on the service or metrics generated internally by your applications via statsd, or JMX. If your monitoring system natively provides network data, you can also use that to create response time-based alerting.

The following example is a public REST API endpoint monitoring alert for latency over 1 second in a 10 minute window, over the `javaapp` deployment in the production namespace `prod`.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/03/kubernetes_service_latency_alert-1.png" alt="Alerting Kubernetes service latency" width="975" height="651" class="alignleft size-full wp-image-4275" />][6] 

All these alerts will highly depend on your application, your workload patterns, etc... but the real question is how you’re going to get the data itself consistently across all your services. In an ideal environment you don’t need to purchase a synthetic monitoring service in addition to your core monitoring tools to get this one critical set of metrics.

Some metrics and their alerts often found in this category are:

*   Service response time
*   Service availability up/down
*   SLA compliance
*   Successful / error requests per second

## Alerting on services running on Kubernetes {#services}

When looking at the service level, shouldn’t be very different from what you were doing before Kubernetes if you had your services clustered. Think of databases like MySQL/MariaDB or MongoDB where you will look at the replication status and lag. Is there anything to take into account now then?

Yes! if you want to know how your service operates and performs globally you will need to leverage your monitoring tool capabilities to do metric aggregation and segmentation based on containers metadata.

We know Kubernetes tags containers within a deployment, or exposed through a service, as we explained in <a href="https://sysdigrp2rs.wpengine.com/blog/monitoring-kubernetes-with-sysdig-cloud/" target="_blank">How to Monitor Kubernetes</a>. Now you need to take that into account when you define your alerts, for example scoping alerts only for the production environment, probably defined by a namespace.

The following is an example from a Cassandra cluster:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/03/kubernetes_cassandra_alert-1.png" alt="Alerting Kubernetes Cassandra" width="973" height="668" class="alignleft size-full wp-image-4269" />][7] 

The metric `cassandra.compactions.pending` exists per instance if we run Cassandra in Kubernetes, AWS EC2 or Openstack, but now we want to look at this metric aggregated across the `cassandra` replication controller inside our `prod` namespace. Both labels have come from Kubernetes, but we could also change the scope including tags coming from AWS or other cloud providers, like availability zones for example.

Some metrics and their alerts often found in this category are:

*   HTTP requests
*   Database connections, replication
*   Threads, file descriptors, connections
*   Middleware specific metrics: Python uwsgi workers, JVM heap size, etc

Also, if external managed services are being used, you most probably want to import metrics from those providers as they still might have incidents that you need to react to.

## Alerting on the Kubernetes infrastructure {#kubernetes}

Monitoring and alerting at the container orchestration level is two-fold. On one side we need to monitor if the services handled by Kubernetes do meet the requirements we defined. On the other side we need to make sure all the components of Kubernetes are up and running.

### Services handled by Kubernetes

#### 1\.1 Do we have enough pods/containers running for each application?

Kubernetes has a few options to handle an application that has multiple pods: Deployments, Replica Sets and Replication Controllers. There are few differences between them but the 3 can be used to maintain a number of instances of running the same application. There the number of running instances can be changed dynamically if we scale up and down, and this process can even be automated with auto-scaling.

There are also multiple reasons why the number of running containers can change: rescheduling containers in a different host because a node failed or because there are not enough resources; a rolling deploying of a new version, etc. If the number of replicas or instances running during an extended period of time is lower than the number of replicas we desire, it’s a symptom of something not working properly (not enough nodes or resources available, Kubernetes or Docker Engine failure, Docker image broken, etc).

An alert that compares: 

    timeAvg(kubernetes.replicaSet.replicas.running) < timeAvg(kubernetes.replicaSet.replicas.desired)
    

across all services is almost a must in any Kubernetes deployment. As we mentioned before, this situation is acceptable during container reschedule and migrations, so keep an eye on the configured `.spec.minReadySeconds` value for each container (time from container start until becomes available in ready status). You might also want to check `.spec.strategy.rollingUpdate.maxUnavailable` which defines how many containers can be taken offline during a rolling deployment.

The following is an example alert with this condition applied to a deployment `wordpress-wordpress` within a `wordpress` namespace in a cluster with name `kubernetes-dev`.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/03/kubernetes_low_replicas_alert-1.png" alt="Alerting Kubernetes low replicas replicaSet" width="975" height="694" class="alignleft size-full wp-image-4273" />][8] 

#### 1\.2 Do we have any pod/containers for a given application?

Similar to the previous alert but with higher priority (this one for example is a candidate for getting paged in the middle of the night), we will alert if there are no containers at all running for a given application.

In the following example, we apply the alert for the same deployment but triggering if running pods is < 1 during 1 minute:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/03/kubernetes_no_running_replicaset_alert-1.png" alt="Kubernetes no replicas replicaSet alert" width="975" height="697" class="alignleft size-full wp-image-4274" />][9] 

#### 1\.3 Is there any pod/container in a restart loop?

When deploying a new version which is broken, if there are not enough resources available or just some requirements or dependencies are not in place, we might end up with a container or pod restarting continuously in a loop. That’s called *CrashLoopBackOff*. When this happens pods never get into ready status and therefore are counted as unavailable and not as running, so this scenario is already captured by the alerts before. Still I like to set up an alert that catches this behavior across our entire infrastructure and lets us know the specific problem right away. It’s not the kind of alert that interrupts your sleep but gives you useful information.

This is an example applied across the entire infrastructure detecting more than 4 restarts over the last 2 minutes:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/03/kubernetes_crashloopbackoff_alert-1.png" alt="Kubernetes CrashLoopBackOff alert" width="974" height="579" class="alignleft size-full wp-image-4270" />][10] 

### Monitoring Kubernetes services

In addition to making sure Kubernetes is doing its job, we also want to monitor Kubernetes internal health. This will depend on the different components on your Kubernetes setup, as these can change depend on your deployment choices but there are some basic components that will definitely will be there.

#### 2\.1 Is etcd running?

etcd is the distributed service discovery, communication, command channel for Kubernetes. Monitoring etcd can go as deep as monitoring any distributed key value database but will keep things simple here: can we reach etcd?

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/03/kubernetes_etcd_alert-1.png" alt="Kubernetes etcd alert" width="977" height="659" class="alignleft size-full wp-image-4271" />][11] 

We can go further and monitor set commands failure or node count, but we will leave that for a future article around etcd monitoring.

#### 2\.2 Do we have enough nodes in our cluster?

A node failure is not a problem in Kubernetes, the scheduler will spawn containers in other available nodes. But, what if we are running out of nodes? Or the resources requirements for the deployed applications overbook existing nodes? Or are we hitting any quota limit?

Alerting in this cases is not easy, as it will depend on how many nodes you want to have on standby or how far you want to push oversubscription on your existing nodes. To monitor node status alert on the metrics `kube_node_status_ready` and `kube_node_spec_unschedulable`.

If you want to alert on capacity, you will have to sum each scheduled pod requests for cpu and memory and then check doesn’t go over each node `kube_node_status_capacity_cpu_cores` and `kube_node_status_capacity_memory_bytes`.

## Alerting on the host / node layer {#host}

Alerting at the host layer shouldn’t be very different from monitoring VMs or machines. It’s going to be mostly about if the host is up or down/unreachable, and resources availability (CPU, memory, disk, etc).

The main difference is the severity of the alerts now. Before, a system down likely meant you had an application down and an incident to handle (barring effective high availability). With Kubernetes, services are now ready to move across hosts and host alerts should never wake up up from bed.

Let’s see a couple of options that we should still consider:

#### 1\. Host is down

If a host is down or unreachable we want to receive a notification. We will apply this single alert across our entire infrastructure. We are going to give it a 5 minutes wait time in our case, since we don’t want to see noisy alerts on network connectivity hiccups. You might want to lower that down to 1 or 2 minutes depending on how quickly you want to receive a notification.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/03/kubernetes_host_down_alert-1.png" alt="Kubernetes host down alert" width="976" height="489" class="alignleft size-full wp-image-4272" />][12] 

#### 2\. Disk usage

This is a slightly more complex alert. We apply this alert across all file systems of our entire infrastructure. We manage to do that setting `everywhere` as scope and firing a separate evaluation/alert per `fs.mountDir`.

This is a generic alert that triggers over 80% usage but you might want different policies like a second higher priority alert with a higher threshold like 95% or different thresholds depending on the file system.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/03/kubernetes_disk_usage_alert-1.png" alt="Kubernetes node disk usage alert" width="993" height="582" class="alignleft size-full wp-image-4278" />][13] 

If you want to create different thresholds for different services or hosts, simply change the *scope* where you want to apply a particular threshold.

#### 3\. Some other resources

Usual suspects in this category are alerts on load, CPU usage, memory and swap usage. You probably want to alert if any of these is significantly high during a prolonged time frame. A compromise needs to be found between the threshold, the wait time and how noise can become your alerting system with no actionable alerts.

If you still want to set up metrics for these resources look at the following metrics:

*   For load: `load.average.1m`, `load.average.5m` and `load.average.15m`
*   For CPU: `cpu.used.percent`
*   For memory: `memory.used.percent` or `memory.bytes.used`
*   For swap: `memory.swap.used.percent` or `memory.swap.bytes.used`

Some people also include in this category monitoring the cloud provider resources that are part of their infrastructure.

## Sysdig bonus: monitoring syscalls {#syscalls}

From the moment an alert triggers and you receive the notification, the real work starts for the members of the DevOps team on duty. Sometimes the *run book* is as simple as checking it was just an minor anomaly in the workload. It might be an incident in the cloud provider, but hopefully Kubernetes is taking care of that and just struggling for a few moments coping with the load.

But if some more hairy problem is in front of us, we can see ourselves pulling out all the weapons: providers status pages, logs (hopefully from a central location), any kind of APM or distributed tracing if developers instrumented the code or maybe external synthetic monitoring. Then we pray the incident left some clues for us in any of these places so we can trace down the problem to the multiple source causes that came along together.

What if we could automatically start a dump of all system calls when the alert was fired? System calls are the source of truth of what happens and will contain all the information we can get. Chances are that the issue still is exposing the source causes when the alert is triggered. <a href="https://www.sysdigrp2rs.wpengine.com/" target="_blank">Sysdig Monitor</a> allows to automatically start a capture of all system calls when an alert gets triggered.

For example I always configure this for *CrashLoopBackOff* scenarios:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/03/kubernetes_sysdig_capture_alert-1.png" alt="Kubernetes sysdig capture with alert" width="797" height="950" class="alignleft size-full wp-image-4276" />][14] 

<a href="https://www.sysdigrp2rs.wpengine.com/" target="_blank">Sysdig Monitor</a> agent exposes metrics that are calculated from system call interception that wouldn’t be possible unless you had instrumented your code. Think of HTTP response time or SQL response time. Sysdig Monitor agent captures `read()` and `write()` system calls on sockets, decodes the application protocol and calculates the metrics that are forwarded into our time series database. The beauty of this is getting dashboards and alerts out of the box without touching your developer’s code:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/03/Peek-2017-06-13-19-46.gif" alt="Create Kubernetes alert with Sysdig Monitor" width="1820" height="926" class="alignleft size-full wp-image-4277" />][15] 

## Conclusions

We have seen how using container orchestration platforms increase the number of pieces moving around in your system. Having container native monitoring in place is a key element for having a reliable infrastructure. Monitoring cannot just focus on the infrastructure later but needs to understand the entire stack from the hosts at the bottom up to the top where the application metrics are.

Being able to leverage Kubernetes and cloud providers metadata to aggregate and segment metrics and alerts will be a requirement for effective monitoring across all layers. We have seen how to use the labels to update the alerts we already had or create the new ones required on Kubernetes.

Next, let’s take a look at a typical [service discovery troubleshooting challenge in a Kubernetes][16]-orchestrated environment.

<iframe width="853" height="480" src="https://www.youtube.com/embed/Y47lYsNuKvA" frameborder="0" allowfullscreen></iframe>

 [1]: #apps
 [2]: #services
 [3]: #kubernetes
 [4]: #host
 [5]: #syscalls
 [6]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/03/kubernetes_service_latency_alert-1.png
 [7]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/03/kubernetes_cassandra_alert-1.png
 [8]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/03/kubernetes_low_replicas_alert-1.png
 [9]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/03/kubernetes_no_running_replicaset_alert-1.png
 [10]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/03/kubernetes_crashloopbackoff_alert-1.png
 [11]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/03/kubernetes_etcd_alert-1.png
 [12]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/03/kubernetes_host_down_alert-1.png
 [13]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/03/kubernetes_disk_usage_alert-1.png
 [14]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/03/kubernetes_sysdig_capture_alert-1.png
 [15]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/03/Peek-2017-06-13-19-46.gif
 [16]: /blog/kubernetes-service-discovery-docker/