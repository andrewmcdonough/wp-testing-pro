---
ID: 5564
post_title: Monitoring Alibaba Container Service
author: Knox Anderson
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/monitoring-alibaba-container-service/
published: true
post_date: 2017-12-18 10:51:54
---
99% of the time HackerNews is an awesome time sink, but every once in awhile something there inspires you to try out a new technology, platform, etc. In this case two fresh slices of pizza and the HN post below were the perfect excuse to try a new cloud platform. We’ll dive into spinning up a container cluster and monitoring Alibaba Container Service.



[<img style="max-width: 100%; height: auto;" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/Screen-Shot-2017-07-13-at-10.36.59-PM.png" alt="Hacker News" width="1200" height="750" class="alignleft wp-image-4350" />][1]

Alibaba Cloud has been around for awhile, but with their new $300 trial credit offer there’s no reason not to spin up some instances to see what it’s about.

Not to be confused with AWS ECS Alibaba refers to their <a href="https://www.alibabacloud.com/product/ecs" target="_blank">compute service as ECS</a> and like GCP, AWS, and Azure they have many services for <a href="https://www.alibabacloud.com/product/apsaradb-for-rds?spm=a3c0i.7911826.675768.dnavproductsd1.289572c4oxcV77" target="_blank">Databases</a>, <a href="https://www.alibabacloud.com/product/server-load-balancer?spm=a3c0i.63466.675768.dnavproductsa3.3f19f412EK5DOD" target="_blank">load balancers</a>, etc. There’s a ton of different free options, but naturally the first thing I wanted to check out was their container service.

## Setting up Alibaba Container Service {#settingupalibabacontainerservice}

The Alibaba Container Service makes it easy to get a handful of hosts booted up, then to schedule and run containerized applications on them. But before we dive into visibility let’s take a look at the building blocks of the Container Service. The first thing we need to do to get started is boot up a cluster.

[<img style="max-width: 100%; height: auto;" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/alibaba-creat-cluster.png" alt="Create Cluster Alibaba" width="1200" height="750" class="alignleft wp-image-4350" />][2]

When building a cluster just click through the GUI to choose your Region, Number of nodes, security group parameters and you’re all set.

[<img style="max-width: 100%; height: auto;" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/image_2.png" alt="Alibaba Container Service Cluster" width="1200" height="750" class="alignleft wp-image-4350" />][3]

Now that you’ve got a small cluster of nodes, let’s deploy some applications. There are a couple different options baked into the Container Service Console

*   Docker Images - Choose any image from the docker public repo and with a single click start that container on your service

*   Orchestration - use docker compose within the console via orchestration templates, or roll <a href="https://kubernetes.io/docs/getting-started-guides/alibaba-cloud/" target="_blank">your own orchestrator like kubernetes</a> or mesos

*   Applications - pre-built application stacks, or applications that you’ve uploaded to run in containers on the nodes in your cluster

I chose to boot up one of the gitlab applications as well as the wordpress service into the cluster we have running.

[<img style="max-width: 100%; height: auto;" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/Screen-Shot-2017-07-29-at-9.03.19-PM.png" alt="Orchestration Services" width="1200" height="750" class="alignleft wp-image-4350" />][4]

## Installing Sysdig Monitor {#installingsysdigmonitor}

Installing Sysdig Monitor in your cluster is <a href="https://www.youtube.com/watch?v=C7NucdQlKCY" target="_blank">crazy fast</a>. Just copy the docker run command or use existing orchestrator facilities like daemonset in kubernetes to get our agent installed. The Sysdig Monitor agent runs as a container on each node, and then loads a non block kernel module via DKMS to start collecting all system, application, network, and custom metrics (statsd, jmx, prometheus) from your system.

[<img style="max-width: 100%; height: auto;" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/alibaba-install.gif" alt="Sysdig Monitor Install" width="1200" height="750" class="alignleft wp-image-4350" />][5]

## Monitoring Alibaba Container Service {#monitoringalibabacontainerservice}

### Monitoring Physical Infrastructure: Hosts & Containers {#monitoringphysicalinfrastructurehostscontainers}

One of the first things to do when you get Sysdig Monitor installed in any environment is to look at a topology map of the cluster. Here we can see every node in the cluster, the cpu usage of the node, and then all ingress and egress connections to other nodes we’re monitoring or external services. 

From here we can drill down even further inside the host and see all the containers, their network connections, and then go another layer deeper by drilling down to the process level or <a href="https://sysdigrp2rs.wpengine.com/blog/the-big-oom-theory/" target="_blank">even syscall</a> of what’s running inside the container. This provides huge value into understanding the performance of Alibaba Container Service, most monitoring tools only provide container metrics via the docker stats API so you know "hey, my container is running at 98% CPU" but you don’t know why, or what’s running inside that container.

[<img style="max-width: 100%; height: auto;" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/alibaba.gif" alt="Alibaba Topology Maps" width="1200" height="750" class="alignleft wp-image-4350" />][6]

[tweet_box design="default" float="none"]How to create dynamic topology maps for Alibaba Container Service[/tweet_box] 

### Monitoring Applications running on Alibaba Container Service {#monitoringapplicationsrunningonalibabacontainerservice}

Next let’s move to the application layer. In the previous gif we had looked at a host and noticed that there was a MySQL process running inside the wordpress container. Sysdig Monitor comes with hundreds of out of the box <a href="https://sysdigrp2rs.wpengine.com/product/monitor/#integrations" target="_blank">integrations with popular applications and services</a>. We’ll automatically recognize the process or protocol and then start pulling metrics from that application and tag them with the metadata from your cloud provider and orchestrator. Such as this example of nginx errors segmented by the kubernetes namespace that they’re coming from.

[<img style="max-width: 100%; height: auto;" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/nginx-errors-alibaba.png" alt="Kubernetes nginx errors" width="1200" height="750" class="alignleft wp-image-4350" />][7]

The same applies to any custom metric from <a href="https://sysdigrp2rs.wpengine.com/blog/how-to-collect-statsd-metrics-in-containers/" target="_blank">statsd</a>, jmx or <a href="https://sysdigrp2rs.wpengine.com/blog/prometheus-metrics/" target="_blank">prometheus</a>. Just install the agent as a container on the node and we’ll discover everything whether it’s running inside a container or on the node. Let’s check out the gif below to see what this looks like for our containerized MySQL application. 

[<img style="max-width: 100%; height: auto;" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/alibaba-Mysql.gif" alt="MySQL dashboard alibaba" width="1200" height="750" class="alignleft wp-image-4350" />][8]

### Exploring our Container Service Environment {#exploringourcontainerserviceenvironment}

Next let’s flip over to the explore page of Sysdig Monitor to slice and dice our infrastructure to dynamically troubleshoot and visualize services. The explore page works like an interactive top or pivot table so you can set up and hierarchy and change it on the fly depending on what questions you want to answer about services, hosts, or containers. Here we can look at hosts first, and then switch our grouping up to get the performance of images and the containers that are sharing that image regardless of what physical infrastructure they’re running on.

[<img style="max-width: 100%; height: auto;" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/alibabaexplore.gif" alt="Explore Alibaba" width="1200" height="750" class="alignleft wp-image-4350" />][9]

## Conclusion {#conclusion}

A couple of slices of pizza and some free credits to learn about monitoring Alibaba Container Service were more than enough to keep me entertained while futzing with different applications on this service.

On the Sysdig Monitor side we’ve still barely touched the surface of the insights you can get into Alibaba Container Service. Check out some of these posts to learn more about how we do <a href="https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/204013259-Alerting-and-Notifications" target="_blank">alerting</a>, <a href="https://sysdigrp2rs.wpengine.com/blog/monitoring-kubernetes-with-sysdig-cloud/" target="_blank">kubernetes monitoring</a>, or more about our <a href="https://sysdigrp2rs.wpengine.com/blog/prometheus-metrics/" target="_blank">custom metrics integrations</a>.

If you’re already running on Alibaba Container Service or any other cloud platform then kick off a <a href="https://sysdigrp2rs.wpengine.com/docker-monitoring" target="_blank">15 node free trial of Sysdig Monitor</a> and let us know what you think!

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/Screen-Shot-2017-07-13-at-10.36.59-PM.png
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/alibaba-creat-cluster.png
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/image_2.png
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/Screen-Shot-2017-07-29-at-9.03.19-PM.png
 [5]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/alibaba-install.gif
 [6]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/alibaba.gif
 [7]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/nginx-errors-alibaba.png
 [8]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/alibaba-Mysql.gif
 [9]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/alibabaexplore.gif