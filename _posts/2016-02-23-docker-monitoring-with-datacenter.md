---
ID: 2428
post_title: 'Docker monitoring with Docker Datacenter &#038; Sysdig'
author: Apurva Dave
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/docker-monitoring-with-datacenter/
published: true
post_date: 2016-02-23 16:17:29
---
Congrats to the Docker team for releasing Docker Datacenter this week! This is yet another solid step forward for building a production-grade, enterprise container ecosystem. The Sysdig team hates to miss a party, so our contribution was to provide deep integration with Docker Universal Control Plane (UCP), a key element in Docker Datacenter. With this new integration you can monitor Docker environments with the same visibility you have across the rest of your IT infrastructure. Today we’re happy to announce general availability of this functionality in our container monitoring platform, Sysdig Cloud. 

## What is Docker DataCenter? 

If you’d like the full story, then there’s no better place to go than the [Docker blog][1] and read the launch post. Here’s the quick summary:

Docker Datacenter is an integrated, end-to-end platform for agile application development and management at any scale. Comprised of Docker UCP, Docker Trusted Registry and embedded support for Docker Engine, Docker Datacenter addresses the requirements for organizations that want to manage the application lifecycle of Dockerized applications from development through production. Enterprises are using Docker Datacenter to deploy an on-premises Containers-as-a Service (CaaS) solution. CaaS is an IT-managed and secured application environment where developers can, in a self-service manner, build and deploy applications. 

It’s been in Beta for quite some time, and now Docker has deemed it ready for prime time. Prime time often means production environments, and when people start moving into production… well, they start thinking about monitoring and troubleshooting. Enter Sysdig. 

## Sysdig Integration with Docker Datacenter

Datacenter is a really powerful concept that allows software operators to be more agile and more cost effective. But, in general, the move to microservices and container-based applications means that some systems that are now manually organized are much more complex. The containers that make up an application or service may be distributed across multiple nodes or multiple clouds, and the containers themselves are black boxes from a application monitoring perspective.

Luckily, Sysdig integrates with Datacenter to provide much-needed monitoring, troubleshooting, and general visibility into these environments. We do this by integrating deeply with container orchestration and management tools (such as UCP) and leveraging the information they possess regarding your dynamic environment.

Our support of Docker Datacenter is actually quite straightforward. Quite some time ago, Sysdig [added support for docker labels][2] in order to provide our users with more context when monitoring docker environments. UCP (and, in fact, Swarm and Compose as well) leverage the same base concept of labels as a way to add metadata to your containers. Labels are generic key value pairs with very few restrictions. Read more about them [here][3].

Docker does preserve a namespace of labels for itself, and UCP uses that to create a set of labels by default. We use the defaults to create smart, relevant views within Sysdig Cloud, and of course let you use your own labels to view custom groups of containers as well.

The most set of labels (other than your own custom labels) are:

<div style="width: 100%; float: left;">
  <pre>com.docker.compose.container-number
com.docker.compose.oneoff
com.docker.compose.project
com.docker.compose.service
com.docker.compose.version
com.docker.swarm.id
com.docker.ucp.access.owner
com.docker.ucp.InstanceID
com.docker.ucp.node
com.docker.ucp.upgrades_from
com.docker.ucp.version</pre>
</div>

By supporting Docker labels, we can automatically understand the proper hierarchy when using Docker orchestration products. The commonly-used label hierarchy for DataCenter is:

<div style="width: 100%; float: left;">
  <pre>com.docker.compose.project > com.docker.compose.service > container.id (or name)
</pre>
</div>

“Project” is essentially an application (and in fact is called that within the UCP UI), whereas “Service” represents a logical sub-component of your application - perhaps a microservice.

The net effect of this is that we can take what was previously a scattered set of containers and organize them by labels into views that make sense for your monitoring and troubleshooting efforts.

Take a look below as we switch from a physical, host-and-container view to a logical, application view just by making one selection:

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/02/feature-Docker-Data-Center-grouping.gif"></a>[<img alt="Docker Monitoring" class="alignnone" height="536" src="/wp-content/uploads/2016/02/feature-Docker-Data-Center-grouping.gif" width="750" />][4] 

## Monitor Docker Datacenter with Sysdig Cloud

Sysdig Cloud leverages open source sysdig technology to provide a best in class Docker monitoring, analytics, alerting, and dashboarding for containerized environments - including scheduling and orchestration tools that enable sophisticated and automated deployment of containers. At a high level, it is as simple as this: with Sysdig Cloud you get full visibility inside every container across your entire operating environment.

With Sysdig Cloud, you can do things like:

1.  Monitor Dockerized applications with no app monitoring plugins needed
2.  Collect service-oriented performance data like top URLs, HTTP error codes, and database queries
3.  Automatically map the topology of your Docker environments to highlight dependencies and bottlenecks
4.  Alert based off your applications and service performance, not just container or host resource utilization

Again, all of this is achieved without needing to instrument your docker containers OR your application code. All you have to do is add the Sysdig Cloud container to your hosts.

As a high-level example, take a look at our topology view, which in this case is measuring response time between logical services in our application. From here you can zoom in on any part of your application – starting at the highest level application or service, and drilling all the way down to a particular process inside a docker container. 

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/02/DDC-1.png"></a>[<img alt="Docker Monitoring" class="alignnone" height="536" src="/wp-content/uploads/2016/02/DDC-1.png" width="750" />][5] 

*
The Sysdig topology view lets you visually monitor Docker Datacenter applications and their relationships.* 

You can also easily see inside your Docker containers to understand what your applications are doing. One quick example of Service-Oriented Performance Management: you can see SQL activity across your database cluster to understand the number of requests, response times, top queries, and slowest queries:

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/02/DDC-2.png"></a>[<img alt="Docker Monitoring" class="alignnone" height="536" src="/wp-content/uploads/2016/02/DDC-2.png" width="750" />][6] 

*
Deep application monitoring within Docker containers… without instrumenting every container.* 

## Pre-built dashboard templates for monitoring UCP

In addition to making it simple to collect relevant data, we just went ahead and created dashboard templates for monitoring Docker UCP.

Here’s a resource-level overview of your system, including top applications by activity:

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/02/Docker-UCP-Dashboard.png"></a>[<img alt="Docker Monitoring" class="alignnone" height="536" src="/wp-content/uploads/2016/02/Docker-UCP-Dashboard.png" width="750" />][7] 

And here’s another focused on monitoring Docker services:

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/02/Docker-Service-Dashboard.png"></a>[<img alt="Docker Monitoring" class="alignnone" height="536" src="/wp-content/uploads/2016/02/Docker-Service-Dashboard.png" width="750" />][8] 

You get these out of the box, no work required. And as your environment changes these dashboards automatically update. Less work for you!

Naturally, we’ll keep improving these dashboards as we learn more about what works best for monitoring docker environments. 

## Adaptive Alerting using Docker Labels

One last thing… our alerting functionality also takes advantage of Docker labels to make it easier to monitor your system. As Docker Datacenter schedules additional containers to scale up, say, your database (db) service, Sysdig automatically applies relevant alerts to those new containers. We call this adaptive alerting.

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/02/DDC-3.png"></a>[<img alt="Docker Monitoring" class="alignnone" height="536" src="/wp-content/uploads/2016/02/DDC-3.png" width="750" />][9] 

In this case, I configured an alert with a particular scope, consisting of labels from Docker Datacenter. As Datacenter spins up more containers, they inherit the correct labels - and by extension the right Sysdig Cloud alerts. This example sets an alert for the “db” (database) within the “voting app” where an alert should fire any time an entity is down… and an entity is designated as a container. You can of course create an alert on a metric value instead of up/down, and scope/segmentation are completely flexible. I just chose a simple example to start with.

## Summary: Monitoring Docker DataCenter with Sysdig

Docker Datacenter simplifies the act of deploying containerized applications into production. Our deep integrations with Docker ensure that your team will be able to confidently move into production with visibility that lets you know how your applications are performing. If you’re thinking about building the capability to monitor, troubleshoot, and analyze your container infrastructure - Docker or otherwise - we hope you’ll consider Sysdig. You can always [start a free trial][10] to see what Sysdig can do for you.

 [1]: https://blog.docker.com/2016/02/docker-datacenter-caas/
 [2]: https://sysdigrp2rs.wpengine.com/sysdig-cloud-adds-support-for-docker-labels/
 [3]: https://docs.docker.com/engine/userguide/labels-custom-metadata/
 [4]: /wp-content/uploads/2016/02/feature-Docker-Data-Center-grouping.gif
 [5]: /wp-content/uploads/2016/02/DDC-1.png
 [6]: /wp-content/uploads/2016/02/DDC-2.png
 [7]: /wp-content/uploads/2016/02/Docker-UCP-Dashboard.png
 [8]: /wp-content/uploads/2016/02/Docker-Service-Dashboard.png
 [9]: /wp-content/uploads/2016/02/DDC-3.png
 [10]: https://sysdigrp2rs.wpengine.com/sign-up/