---
ID: 10340
post_title: 'Sysdig Monitor 3.0 &#8211; Enterprise-grade Prometheus, Kubernetes insights and more.'
author: Eric Carter
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-monitor-3-0-enterprise-grade-prometheus/
published: true
post_date: 2018-09-26 01:38:10
---
Today we [announced][1] the launch of enterprise-grade Prometheus monitoring with Sysdig Monitor 3.0. We've added new Prometheus capabilities like PromQL, a Grafana plugin and new enhancements for our already rich Kubernetes monitoring. If you love Prometheus like we do, and especially if your cloud environment is growing quickly, read on to learn more about what we’re doing with Prometheus, Kubernetes and more.

## **Why embrace Prometheus?** {#why-embrace-prometheus}

Let's tackle this one right up front. We’ve embraced [Prometheus monitoring][2] for two key reasons:

<ol style="list-style-type: decimal;">
  <li>
    <p>
      To help Sysdig Monitor users take advantage of the rich metrics and capabilities Prometheus brings to monitoring - especially for Kubernetes, Docker and microservices. <em>Prometheus makes Sysdig that much better.</em>
    </p>
  </li>
  
  <li>
    <p>
      To help Prometheus customers meet requirements in large, complex production environments with things like backend scale, broad telemetry and event data, long-term data retention, access control and more. <em>Sysdig makes Prometheus that much better.</em>
    </p>
  </li>
</ol>

Simply put, we love Prometheus! With the work we’ve been doing around Prometheus, I'm happy to report we’ve started to contribute to the open source project in addition to providing capabilities for enterprise customers.

## **New Prometheus monitoring, Kubernetes and usability features** {#new-prometheus-monitoring-kubernetes-and-usability-features}

As you transition to full-scale production with Kubernetes, Docker or similar, you’re likely to face a new set of requirements for monitoring and security – perhaps some you haven’t anticipated. Sysdig Monitor 3.0 is designed to help Prometheus, Kubernetes and Docker users meet these needs. This includes:

*   **Scalability:** To scale monitoring with a rapidly growing environment with high-churn, high-volume metrics. 

*   **Long(er)-term retention of data:** To analyze service trends and metrics over time.

*   **Multi-cluster, multi-cloud visibility: **To monitor, compare, and correlate services across cluster deployments and clouds.

*   **Capacity planning:** To make informed, intelligent choices about where and when to increase or decrease cluster resources based on factors like oversubscription.

*   **Kube components health/state monitoring:** To ensure the foundation of your orchestrated environment is up and performing well.

*   **Access control and isolation:** To limit environment and data access to specific users, groups and teams.

*   **High availability: **To ensure uptime with redundancy and automatic failover.

Next, what I want to do is give you a rundown of what's new with Sysdig Monitor 3.0. We're writing additional blogs to follow that dive deeper into each of these areas.

[tweet_box design="default" float="none"]Sysdig Monitor 3.0 adds enterprise-grade #Prometheus monitoring, new integrations, and even richer #Kubernetes and #Docker monitoring.[/tweet_box] 

### For Prometheus users: Embracing Prometheus metrics, PromQL, Grafana {#for-prometheus-users-embracing-prometheus-metrics-promql-grafana}

Sysdig Monitor automatically collects Prometheus metrics and offers a powerful way to aggregate, store, alert, visualize – and now query Prometheus data. Sysdig Monitor 3.0 extends what’s possible with Prometheus by adding [Prometheus Query Language (PromQL)][3] compatibility. PromQL lets users select and aggregate time series data in real time using expressions that can be displayed in dashboards, graphs or tables. Sysdig extends PromQL *beyond* Prometheus metrics! Now DevOps and IT operations teams will be able to calculate advanced metrics to meet the unique needs of their enterprise using *any* collected metric or event including Prometheus, JMX, StatsD, application, system and orchestration metrics.

Another feature of Sysdig Monitor 3.0 intended for the Prometheus community is the addition of a [Grafana plugin][4]. Now you can add Sysdig as a Grafana data source to visualize your Sysdig Monitor metrics and dashboards in Grafana.

<center>
  <a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/sysdig-prometheus-service-golden-signals.png"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/sysdig-prometheus-service-golden-signals-1170x970.png" alt="sysdig prometheus grafana" width="640" height="531" class="aligncenter wp-image-10521 size-large" /></a>
</center>

Ultimately what we’ve provided for Prometheus users with Sysdig Monitor 3.0 is a way to solve monitoring challenges at scale with minimum disruption to the tools and workflows they know and love.

Here are a couple of resources I recommend reading for a full rundown of Sysdig + Prometheus:

*   Whitepaper: [12 Ways Sysdig Makes Prometheus Monitoring Enterprise Ready][5]

*   Customer stories: [How 5 Companies use Prometheus and Sysdig for Monitoring.][6]

### For Kubernetes users: Universal Kubernetes support, cluster management dashboards and StatefulSet metrics {#for-kubernetes-users-universal-kubernetes-support-cluster-management-dashboards-and-statefulset-metrics}

Beyond Prometheus capabilities, Sysdig Monitor 3.0 brings several advancements that will be important no matter which distribution of Kubernetes you use. Not only are we expanding the Kubernetes insights Sysdig Monitor provides, we’re also ensuring you can apply those insights across any Kubernetes environment.

<p id="universal-kubernetes-support">
  <strong>Universal Kubernetes support: </strong>The popularity and effectiveness of Kubernetes for orchestrating containers has given rise to a number of solutions both for on premises use and in the public cloud. Sysdig Monitor 3.0 brings the concept of "Universal Kubernetes support" from Sysdig. What this means is that we’ve verified the use of the Sysdig Container Intelligence Platform with all of the products you can think of including:
</p>

*   Open source Kubernetes

*   OpenShift

*   Kubernetes with Mesosphere DC/OS

*   Kubernetes with Docker 2.0

*   Pivotal Kubernetes Service (PKS)

*   Rancher

*   IBM Cloud Kubernetes Service

*   Amazon Elastic Container Service for Kubernetes (Amazon EKS)

*   Azure Container Service (AKS)

This announcement is intended to give users of any of these solutions the confidence that they can use Sysdig solutions for their solution of choice with all of the benefits of this and previous capabilities we’ve announced to simplify monitoring Kubernetes.

<p id="cluster-management-dashboards">
  <strong>Cluster Management Dashboards: </strong>With more and more services migrating to Kubernetes, monitoring the health and state of your clusters becomes critical to keeping your services up and available. Sysdig Monitor 3.0 delivers new default dashboards to help keep an eye on your Kubernetes platform - and to know when it’s time to add or change the resources you have in place. New cluster state, master health, and capacity management dashboards are designed to increase your operational efficiency and ensure the health of your orchestration system.
</p>

<p id="statefulset-metrics-and-dashboards">
  <strong>StatefulSet Metrics and Dashboards: </strong>Are you deploying stateful apps to your Kubernetes cluster? We’re seeing it more and more (see <a href="https://go.sysdigrp2rs.wpengine.com/docker-usage-report" target="_blank" rel="noopener">Docker usage report</a> for details from earlier this year). New support for Kubernetes <a href="https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/" target="_blank" rel="noopener">StatefulSet</a> metrics and default dashboards with Sysdig Monitor 3.0 helps you monitor for your applications like databases. With the added visibility, our goal is the help you streamline problem identification and resolution for these critical resources.
</p>

### For all Sysdig Monitor users: Streamlined event feed, dashboard templates and Grafana integration {#for-all-sysdig-monitor-users-streamlined-event-feed-dashboard-templates-and-grafana-integration}

<p id="event-feed">
  <strong>Event feed: </strong>As your environment scales to new heights, one of the things you will value is a way to quickly identify the things that deserve your attention. We’ve been working with our customers to design the best way to display consolidated event details. The result is our new streamlined event feed page. With it you will see at-a-glance, consumable, summarized event details coming from across your apps and infrastructure – from Docker and Kubernetes to your CI/CD build and security incidents. With the new event feed you’ll be able to save valuable time and effort when identifying and resolving trouble in the environment.
</p>

<center>
  <a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/event-feed2.png"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/event-feed2-1170x1033.png" alt="" width="640" height="565" class="alignnone wp-image-10790 size-large" /></a>
</center>

<p id="dashboard-templates">
  <strong>Dashboard templates:</strong> Dashboards are great! Everybody’s got one - but one thing we’ve identified is the need for "templates" to help simplify use across different groups and scopes of a container or cloud environment. Sysdig Monitor 3.0 introduces a new concept of “dashboard templates” for Sysdig customers. Now, Sysdig Monitor administrators will save time for internal users by providing a set of dashboards as a service. Pre-canned dashboards will, for instance, provide best practice views for a given aspect of the environment, and users can customize further by choosing from a set of available scope variables.
</p>

<p id="grafana-plugin">
  <strong>Grafana plugin: </strong>Yes, I know I mentioned Grafana above - but I want to take credit for it here again as I highlight usability improvements. The bottom line here is – whether you’re using Prometheus or not – if you love the Grafana interface, you can use it with Sysdig Monitor!
</p>

## Conclusion {#conclusion}

Sysdig Monitor 3.0 will be rolled out to all Sysdig customers in this coming quarter. With it, our goal is to help enterprise customers see a whole and complete picture of their microservices and to help them resolve issues more quickly.

If you’re just getting started, one of the things you’ll discover is that successful monitoring of business-critical applications and infrastructure based on cloud-native solutions like Kubernetes and Docker requires a new approach. If you’re already there, you know what I’m talking about. I expect things like Prometheus to be a part of your journey and as you transition to full-scale production, I think you’ll find that what Sysdig provides will help you deliver efficient, secure applications at enterprise scale and accelerate your time-to-value with Prometheus and Kubernetes.

 [1]: https://sysdigrp2rs.wpengine.com/press-releases/sysdig-monitor-3-0-prometheus-kubernetes-docker
 [2]: https://prometheus.io/
 [3]: https://prometheus.io/docs/prometheus/latest/querying/basics/
 [4]: https://github.com/draios/grafana-sysdig-datasource
 [5]: https://go.sysdigrp2rs.wpengine.com/l/231542/2018-09-14/cjlb8
 [6]: https://go.sysdigrp2rs.wpengine.com/l/231542/2018-09-26/ckcw8