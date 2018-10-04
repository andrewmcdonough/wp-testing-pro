---
ID: 7250
post_title: 2018 Docker Usage Report.
author: Eric Carter
post_excerpt: "Our 2nd annual Docker Usage Report provides insight into real-world customer container deployments over the past year. Read about increasing densities, diversity in container runtimes, and details on what's happening with orchestrators like Kubernetes."
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/2018-docker-usage-report/
published: true
post_date: 2018-05-29 06:00:45
---
Our 2018 Docker Usage Report provides an inside look at shifting container trends as revealed by a point-in-time snapshot of real-world container usage as reported by the [Sysdig Monitor][1] and [Sysdig Secure][2] cloud service.

The quick summary: *Organizations are getting more bang for their hardware buck by packing in 50% more containers per host, Docker still rules the roost but brand name container runtime environments are making inroads, and Kubernetes is still the king of container orchestration.*

Our second annual Docker usage report show tremendous momentum across the Docker container ecosystem year-over-year. As more organizations transition to DevOps and microservices models, and advance expertise in the "modern stack", we see more activity and more scale, but also an increased need for understanding all of the pieces that work well together.

Our study sample includes a broad cross-section of vertical industries and companies ranging in size from mid-market to large enterprises across North America, Latin America, EMEA, and Asia Pacific. Data from 90,000 containers – twice the sample size of last year – provides unique insight into use of containers in production.

[tweet_box design="default" float="none"]What's new with Docker in production? See real-world #container usage through #monitoring and #security metrics by @sysdig[/tweet_box]

What follows is a summary of the key findings. For full details, download your own copy of the report [here][3].

## The top 12 application components running in containers {#top12appcomponents}

***Key Assessment: The old merges with the new*** [caption id="figure1" align="alignnone" width="889"]

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/top-apps.png" alt="top docker apps" width="889" height="280" class="" /> **Figure 1. 12 most popular application components**[/caption] 
Sysdig’s auto-discovery mechanism – [ContainerVision™][4] – shows what processes are used inside containers. Users are consistently utilizing open source solutions to construct their microservices and applications. <a href="https://en.wikipedia.org/wiki/Java_virtual_machine" target="_blank" rel="noopener">Java Virtual Machines (JVM)</a> top the list. Java has been relied on for app services in the enterprise before containers arrived, and now the two – Java and containers – come together as organizations adopt a modern day delivery model.

In addition, increased usage of database solutions like PostgreSQL and MongoDB running in containers signal that the move is on to stateful services in containers. The ephemeral nature of containers left many concerned about running services that collect valuable corporate data in containers. The concern appears to be easing as the data suggests customers are beginning to move to environments completely driven by containers.

## Container density ratchets up {#containerdensityratchetsup}

***Key assessment: Median container density per host rises 50% year-over-year.*** [caption id="figure2" align="alignnone" width="801"]

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/container-density.png" alt="container density" width="801" height="189" class="" /> **Figure 2. In 2017 the median number of containers per host was 10. In 2018 the number rose to 15.**[/caption] 
One of the catalysts for the transition from bare-metal and VM-centric environments to containers is the promise of more efficient utilization of server resources. Compared to <a href="https://go.sysdigrp2rs.wpengine.com/2017-docker-usage-report" target="_blank" rel="noopener">our 2017 report</a>, the median number of containers per host per customer climbed 50%, from 10 to 15.

At the other end of the spectrum, in this survey we saw a customer running **154** containers on a single host! This is up from 95 that we observed last year. That’s a lot of containers. [caption id="figure3" align="alignnone" width="544"]

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/154-containers.png" alt="154 containers" width="544" height="216" /> **Figure 3. Max density observed: 154 containers**[/caption] 
## What container runtimes are in use? {#whatcontainerruntimesareinuse}

***Key assessment: Docker still reigns, but we’re seeing what might be the first signs of cracks in the dam.*** [caption id="" align="alignnone" width="855"]

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/container-runtimes.png" alt="container runtimes" width="855" height="174" class="" /> **Figure 4. Container runtimes: Docker leads, followed by rkt and Mesos.**[/caption] 
<a href="https://www.docker.com/" target="_blank" rel="noopener">Docker</a> shows up the most in production. We didn’t report on other container runtime details in 2017 because, at the time, Docker represented nearly 99% of containers in use. With the recent <a href="https://www.redhat.com/en/blog/coreos-bet" target="_blank" rel="noopener">acquisition of CoreOS by Red Hat</a> (the maker of rkt), and programs like the <a href="https://www.opencontainers.org/" target="_blank" rel="noopener">Open Container Initiative (OCI)</a>, which seeks to standardize container runtime and image specifications, we wanted to take a fresh look to see if container runtime environments are shifting.

In fact they are. In the last year, customers have increased their use of other platforms. Rkt grew significantly to 12%, and Mesos containerizer to four percent. LXC also grew, although at a significantly lower rate. It appears from the data that customers have a greater comfort level with using "non-Docker" solutions in production.

## Lifespan of containers and services {#lifespanofcontainersandservices}

***Key assessment: 95% of containers live less than a week*** [caption id="figure5" align="alignnone" width="490"]

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/container-lifespan.png" alt="container lifespan" width="490" height="365" class="" /> **Figure 5. The majority of containers have lifespan of less than a week.**[/caption] 
With so much industry discussion about the ability to spin services up and down quickly, we decided to look at the lifespan of containers, container images and container-based services. *Just* *h**ow long do containers and services live?*

The following chart highlights the percentage of containers that appear and disappear over different intervals up to and beyond one week.

Eleven percent of containers stay alive for less than 10 seconds. The largest percentage – 27% – are containers that churn between five to 10 minutes.

Why do so many containers have such short lifespans? We know many customers have architected systems that scale as needed with demand and live only as long as they add value. Containers are created, do their work, and then go away. As an example, one customer spins up a container for each job they create in Jenkins. They test the change, then shut down the container. For them this takes place thousands of times a day.

We also looked at how long container images were in use. By looking at this data, we get an idea of how often customers are doing new deploys of updated containers as a part of their DevOps CI/CD process. [caption id="figure7" align="alignnone" width="730"]

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/container-images-lifespan.png" alt="container images lifespan" width="730" height="332" class="" /> **Figure 6. Nearly two-thirds of container images are updated each week.**[/caption] 
Taken altogether, 69% of images are updated in the span of a week.

We also asked, *"What is the lifespan of a service?"*In Kubernetes, the service abstraction defines a set of Pods that deliver a specific function and how to access them. Services allow pods to die and replicate without impacting the application. For example, a cluster may run a Node.js JavaScript runtime service, a MySQL database service, and an NGINX front end service. [caption id="figure7" align="alignnone" width="700"]

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/container-services-lifespan.png" alt="container services lifespan" width="700" height="345" class="" /> **Figure 7. Most container-based services stay up beyond a week.**[/caption] 
Here we see that a majority of services – 67% – live beyond a week. For most customers the goal is to keep applications working around the clock. Containers and pods may come and go, but it is expected that services are up and available.

## Orchestrators for Docker containers {#orchestratorsfordockercontainers}

***Key assessment: First place goes to Kubernetes, followed by Kubernetes and then Kubernetes.*** [caption id="" align="alignnone" width="923"]

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/orchestrators-in-use.png" alt="Orchestrators in use" width="923" height="260" /> **Figure 8. Kubernetes and Swarm grow orchestrator share, Mesos shrinks.**[/caption] 
Sysdig [ServiceVision][5][™][5] automatically identifies orchestrators in use, correlating logical infrastructure objects with container metrics. This awareness tells us what orchestrators are deployed by customers. In the 2018 study <a href="https://kubernetes.io/" target="_blank" rel="noopener">Kubernetes</a> retained the hold on the lead position for the most frequently used orchestrator. No real surprise there as the past year has seen the market evolve and Kubernetes embraced seemingly across the board.

For example, Microsoft uses Kubernetes for its <a href="https://azure.microsoft.com/en-us/services/container-service/kubernetes/" target="_blank" rel="noopener">Azure Kubernetes Service (AKS)</a>, as does IBM with its <a href="https://www.ibm.com/cloud/container-service" target="_blank" rel="noopener">Cloud Container Service</a> and [Cloud Private][6] offering. Even Docker and Mesosphere have added support and functionality for Kubernetes. This means clear lines of demarcation no longer exist as they did in previous years. For instance, Mesosphere is able to deploy and manage "Kubernetes-as-a-service" in a DC/OS environment. Multiple Kubernetes clusters may be deployed under a single Mesosphere cluster.

For our report, we didn’t identify where customers use both orchestrators, but we plan to delve into that in the future.

Docker Swarm climbed into the second slot in this year’s study, surpassing Mesos-based tools. Given Docker has embraced Kubernetes, we didn’t expect this. Possible drivers include:

1.  Swarm’s barrier to entry is incredibly low. While it may not have all the features of Kubernetes, as more people start with containers this may be the first stop in orchestration.

2.  Docker <a href="https://www.docker.com/enterprise-edition" target="_blank" rel="noopener">Enterprise Edition</a>, featuring the Docker Universal Control Plane (UCP) graphical user interface, simplifies many operational aspects of getting started with Swarm. Since Docker’s Kubernetes tie-in came late in 2017, any change in adoption in our customer base from Swarm to Kubernetes might be still forthcoming.

Figure 8. Kubernetes and Swarm grow orchestrator share, Mesos shrinks.

## Cluster size influences orchestrator choice {#clustersizeinfluencesorchestratorchoice}

***Key assessment: Mesos owns the big cluster game.*** [caption id="" align="alignnone" width="904"]

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/relative-cluster-size-3.png" alt="relative cluster size" width="904" height="372" class="" /> **Figure 9. Mesos clusters 50% larger than Kubernetes. Swarm 30% smaller.**[/caption] 
Besides looking at the number of customers using each orchestrator, we asked, "Does cluster size influence which orchestrator an organization might choose?"

While Mesos-based orchestration, including <a href="https://mesosphere.github.io/marathon/" target="_blank" rel="noopener">Mesos Marathon</a> and <a href="https://dcos.io/" target="_blank" rel="noopener">Mesosphere DC/OS</a>, dropped to third in this study, where Mesos is used, the median number of containers deployed is 50% higher than Kubernetes environments. This makes sense given Mesos tends to be targeted at large-scale container and cloud deployments. So while fewer in number, Mesos clusters are typically enterprise-scale.

[Swarm][7] clusters, conversely, were 30% smaller compared to Kubernetes.

## Top flavors of Kubernetes {#topflavorsofkubernetes}

***Key assessment: Here come the Kubernetes distributions.*** [caption id="" align="alignnone" width="878"]

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/kubernetes-based-orchestrators.png" alt="kubernetes-based orchestrators" width="878" height="154" class="" /> **Figure 10. Open source Kubernetes most used, followed by OpenShift and Rancher distributions.**[/caption] 
This year, we dissected the use of Kubernetes by "brand," looking to see if the flavor of Kubernetes in-use was the upstream open source version, or a package provided by a specific vendor. We found that open source Kubernetes continues to hold the lion-share, but it appears that <a href="https://www.openshift.com/" target="_blank" rel="noopener">OpenShift</a> is making inroads and <a href="https://rancher.com/" target="_blank" rel="noopener">Rancher</a> has made some gains as well.

Anecdotally, at Sysdig we see a greater percentage of our on-prem customers, who tend to be larger enterprises running Sysdig solutions in private data centers, adopt OpenShift in greater numbers than customers of our SaaS offering.

Rancher Labs emerged in 2015 with support for both Docker Swarm and Kubernetes. It wasn’t until 2017 that Rancher fully embraced Kubernetes as its orchestrator of choice.

## Most popular alert conditions when using Docker containers {#mostpopularalertconditionswhenusingdockercontainers}

***Key assessment: It’s all about performance and uptime*** [caption id="" align="alignnone" width="460"]

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/alert-conditions.png" alt="kubernetes-based orchestrators" width="460" height="404" class="" /> **Figure 11. Top alerts focus on response time and uptime of application services.**[/caption] 
What keeps container administrators up at night? One way to see is to examine the types of alerts they configure in their container environments.

Responsiveness of app services is at the top of the list. Users want to know, "Is my app performing badly?" To see if a service is running well – or not – users look for the four “<a href="https://landing.google.com/sre/book/chapters/monitoring-distributed-systems.html#xref_monitoring_golden-signals" target="_blank" rel="noopener">Golden Signals</a>” – latency, traffic, errors, and saturation. Sysdig provides Golden Signal dashboards featuring these metrics to help answer the questions:

*   How long does it take to service a request?
*   How much demand is being placed on the system?
*   How often do requests fail?
*   How constrained are system resources?

With this knowledge, users have a good idea of whether the user experience is good or degraded.

Response time is the most widely used alert type configured, closely followed by uptime/downtime alerts. Sysdig allows thresholds so temporary blips, that are well-managed by a well-orchestrated environment, don’t result in false alarms.

Tried and true resource metrics – cpu, memory, and disk usage – are still widely used, with host-based alerts being the most frequently set. Users want to know if the server hosting Docker (physical, VM or cloud instance) is under strain or reaching capacity. The trigger for these alerts is most often set between 80-95% utilization.

On the rise, however, are container-focused resource alerts. The top used alerts come mainly in two flavors, 1) resource utilization, and 2) container count.

By default, containers have no resource limits. Given customers are increasingly alerting on container limits implies they are using Docker runtime configurations to control how much memory, CPU, or disk I/O containers can use and want to know when that goes out of scope and puts application performance at risk.

For container count, the concern is typically tied to the fact that users want at least X number of containers of a given type up and functioning to deliver the required service levels, especially in microservices deployments. For example, "I know my app works well when at least three NGINX containers are up. Anything less and I want to know."

Orchestration-focused alerts are also increasingly popular. "Pod Restart Count" tops the list. In a pod, one or more containers are co-located and co-scheduled, typically as a part of a microservice. If a pod restarts too frequently, it indicates a problem that is likely to impact application performance.

Kubernetes administrators often use event-based alerts as well. This differs from metric-based alerts in that Sysdig looks for event messages generated in the environment such as a Kubernetes "[CrashLoopBackoff][8]" condition, where pods fail and restart repeatedly, or “Liveness probe failed,” which indicates whether a container is alive and running.

Http errors rounds out the list of top alert conditions. Http errors can indicate a problem with software or infrastructure that will ultimately impact performance.

## Popular alert scopes {#popularalertscopes}

***Key assessment: Users want to know – How’re my pods doing?*** [caption id="" align="alignnone" width="878"]

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/alert-scoping.png" alt="kubernetes-based orchestrators" width="878" height="154" class="" /> **Figure 12. Kubernetes pod and namespace rises to top of alert scoping in 2018.**[/caption] 
Alerts are not a one-size-fits-all approach. Alerts can be set or "scoped" for a subset of the environment – be it logical or physical entities – or for the entire infrastructure. Scoping targets any tag or label collected and displayed in the Sysdig solution.

In the 2018 study the most common tags used to scope an [alert are tied to Kubernetes][9]. Scoping by pods is the leading choice followed closely by namespace. That’s not to say that physical hosts don’t matter. Our customers say they do. This is revealed by the fact that scoping by host names and tags is a solid third when it comes to choosing an alert scope.

Container specific scoping is also popular, evenly split across container name, container image, and container ID. Cloud provider tags in 2018 again rank high on the list, frequently targeting "name," “environment,” “ID,” and “region” tags to scope by resource, dev/test and production, application, and location of cloud data center.

## Custom metrics for application and infrastructure monitoring {#custommetricsforapplicationandinfrastructuremonitoring}

***Key assessment: There’s no one custom metrics format to rule them all*** [caption id="" align="alignnone" width="878"]

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/sysdig-custom-metrics.png" alt="sysdig custom metrics" width="878" height="314" class="" /> **Figure 13. JMX is the most used custom metric format.**[/caption] 
Sysdig’s unique instrumentation model automatically collects, without manual configuration, custom metrics like [JMX][10], [StatsD][11], and [Prometheus][12]. This gives us insight into the metrics Developers rely on to monitor various aspects of their applications – like request time and heap usage. We asked the question – "Of customers running containers in their environments, what percentage are using custom metrics, and which ones?"

JMX metrics associated with Java applications were used by 55% of Sysdig SaaS users. This aligns with the fact that we see Java apps are very widely deployed. StatsD comes in at 29% and Prometheus is used by 20% of our SaaS users. With the [popularity of Prometheus][13] and its avid community support, we expect this number to grow over time.

## Popular container registries {#popularcontainerregistries}

***Key assessment: It’s a split decision. Registries are critical but there’s no clear leader*** [caption id="figure1" align="alignnone" width="491"]

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/container-registries-used.png" alt="top docker apps" width="491" height="312" class="" /> **Figure 14. Container registry use is divided across public and private solutions.**[/caption] 
This year we again identified registries-in-use. Of the top 3, <a href="https://cloud.google.com/container-registry/" target="_blank" rel="noopener">Google Container Registry (GCR)</a> has the largest percentage, [Quay][14] is the second most used, followed by Docker and Amazon <a href="https://aws.amazon.com/ecr/" target="_blank" rel="noopener">Elastic Container Registry (ECR)</a>. GCR and ACR are both fully-managed cloud-based private Docker container registries. Quay and Docker can be used either as on-premises solutions or run in the cloud. One caveat for these numbers – only a subset of the user base,  just over 50% provided a clear indication of their registry solution.

## Summary: Momentum and maturity continue with the new stack {#summarymomentumandmaturitycontinuewiththenewstack}

The data in this year’s report provides a point of visibility into the momentum behind the solutions that help customers as they build microservices-based services using containers and modern DevOps practices. New approaches are maturing and helping organizations develop applications more quickly to solve real business challenges and compete in the digital marketplace. We’ll be back next year to share what’s changed in the fast moving Docker space.

#### **Would you like a downloadable PDF version of this report? [Grab it here.][3]**

 [1]: https://sysdigrp2rs.wpengine.com/product/monitor/
 [2]: https://sysdigrp2rs.wpengine.com/product/secure/
 [3]: https://go.sysdigrp2rs.wpengine.com/docker-usage-report
 [4]: https://sysdigrp2rs.wpengine.com/product/how-it-works/#container-vision
 [5]: https://sysdigrp2rs.wpengine.com/product/how-it-works/#service-vision
 [6]: https://www.ibm.com/cloud/private
 [7]: https://docs.docker.com/engine/swarm/
 [8]: https://sysdigrp2rs.wpengine.com/blog/debug-kubernetes-crashloopbackoff/
 [9]: https://sysdigrp2rs.wpengine.com/blog/alerting-kubernetes/
 [10]: http://www.oracle.com/technetwork/articles/java/javamanagement-140525.html
 [11]: https://github.com/etsy/statsd
 [12]: https://prometheus.io/
 [13]: https://sysdigrp2rs.wpengine.com/blog/prometheus-monitoring-and-sysdig-monitor-a-technical-comparison/
 [14]: https://quay.io/