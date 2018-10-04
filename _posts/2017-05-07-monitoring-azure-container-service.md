---
ID: 4038
post_title: 'Monitoring Azure container service with Sysdig: A step-by-step guide'
author: Dan Papandrea
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/monitoring-azure-container-service/
published: true
post_date: 2017-05-07 22:46:51
---
Sysdig hearts (can I emoji here?) monitoring Azure Container Service just as much Amazon or other public cloud providers… and we can prove it to you! 

Azure has made leaps and bounds progress in terms of container and container orchestration support. This blog post will show you some great things you can do with Sysdig and monitoring Azure in terms of giving you a full picture of your container or micro service based applications. 

### Azure container and orchestrator setup



Create your container service in azure with your favorite orchestrator (Sysdig support(s) all 3 of them). (great article for the step by step: [docs.microsoft.com/en-us/azure/container-service/container-service-deployment][1] 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure1.png" width="976" height="618" class="alignleft size-full wp-image-4062" alt="Azure container console" />][2] 



The concept of a resource group is a good one it groups your resources in easy to find “buckets” for deploying and troubleshooting and overall ease of use vs the independent method in Azure Portal Classic and other public cloud UI’s beyond creating automation scripts. I created (2) Azure Container Services, using Docker Swarm and DC/OS Mesosphere as my orchestrators. Azure’s quickstart templates can also be saved as JSON files for this process to be recreated or modified quickly. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure2.png" width="976" height="618" class="alignleft size-full wp-image-4062" alt="DCOS/Swarm Resources" />][3] 



For this exercise, I created a DCOS and Docker Swarm Instance. Kubernetes is also supported by Azure Container Services. 

Here is a high level CPU/Disk Read and Write/Network Azure infrastructure based dashboard of the docker swarm and DCOS masters. Azure’s dashboarding/metrics does a great job of showing me my azure services and infrastructure, along with resource groups and billing.



[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure3.png" width="976" height="618" class="alignleft size-full wp-image-4062" alt="Azure container console" />][4] 



### Deep container monitoring for Azure



What if I really want to get in depth details about monitoring my microservices? Maybe I want to see how my individual containers or microservices are interacting. Perhaps I even want to see http request/response, network connectivity between the containers, and how my front end web services are communicating with backend data services? Top connections/ports connecting to my containers? Maybe I want container CPU shares, Memory, or Disk space on my service along with my overall Azure Container Service.. How would I get more in depth microservices/container visibility within Azure?



That’s where Sysdig comes in and can help you. Sysdig is the container monitoring specialist. We focus on the complex task of seeing inside containers, and then relating that information to your orchestrator and your cloud in real-time. With that, let’s put Sysdig to work on monitoring Azure.

****

### The Sysdig steps for setting up Azure monitoring

**** 

#### For Docker 

All you have to do is deploy the sysdig monitoring agent into your cluster - no setup or configuration needed.. see specific instructions here.. <https://support.sysdigcloud.com/hc/en-us/articles/204498905-Sysdig-Install-Standard-Linux-Docker-CoreOS-> ****

#### For Kubernetes

**** 

We can deploy our agent via daemonset. Microsoft has documentation on the steps to accomplish this: <https://docs.microsoft.com/en-us/azure/container-service/container-service-kubernetes-sysdig> ****

#### For Mesosphere

**** 

You would install the agent on your master instances with the instructions above, and then deploy the agent via our Universe deployment, here are the steps straight from Microsoft on the topic [https://docs.microsoft.com/en-us/azure/container-service/container-service-monitoring-sysdig ][5] 

#### Done… That’s it. 

Sysdig will then automatically collect metadata and other goodies from the Orchestrator of your choice and then help you organize and group your applications for your teams to monitor (container or microservices) in a more holistic way. Sysdig’s magic is, with this simple instrumentation method, you can even see what your applications are doing inside your containers. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure4.png" width="976" height="618" class="alignleft size-full wp-image-4062" alt="Sysdig Monitor Azure" />][6] 



Here is a service performance of a simple MySQL Demo docker compose service running on my Azure Container Services Docker Swarm cluster. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure5.png" width="976" height="618" class="alignleft size-full wp-image-4062" alt="Sysdig Monitor Azure" />][7] 



Here is a topology view of response times for your services running in your docker swarm cluster. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure6.png" width="976" height="618" class="alignleft size-full wp-image-4062" alt="Azure Topology Map" />][8] 



Here’s the MySQL Demo service with a client communicating with a backend MySQL server. We are showing service bottleneck details and response times. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure7.png" width="976" height="618" class="alignleft size-full wp-image-4062" alt="Swarm Monitoring" />][9] 



We are even getting my service’s MySQL(and [many other application integrations][10]) without installing an agent directly into my container) 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure8.png" width="976" height="618" class="alignleft size-full wp-image-4062" alt="Swarm MySQL Monitoring" />][11] 



Sysdig even captures event data if one of your containers goes down right from the Orchestrator… we make your Azure container service even more fantastic. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure9.png" width="976" height="618" class="alignleft size-full wp-image-4062" alt="Docker Event Monitoring" />][12] 



### How does Sysdig do it?

Container and Service Vision that’s how! We automatically find and poll your orchestrator for metadata about your deployment. This allows us to then aggregate your monitoring data on-the-fly to give you smarter, microservice-based views of your resources, instead of just physical host/ip/container views.We were also built to understand microservices from day one. We leverage the data in your orchestration system through a functionality we call ServiceVision to allow us to have an always-up-to-date view of where your containers are deployed and what they are supposed to be doing. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure10.png" width="976" height="618" class="alignleft size-full wp-image-4062" alt="Sysdig" />][13] 



Take a look at all these (OOTB) Dashboards you can create quickly…. All the usual suspects you can deploy with Azure Container Services (Docker/Swarm, Kubernetes, and Mesosphere and Marathon) from an overview to individual services/tasks these templates are there right when you sign up for a trial or sign up for Sysdig. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure11.png" width="976" height="618" class="alignleft size-full wp-image-4062" alt="Swarm Monitoring" />][14] 



Our agent runs at the kernel level for Sysdig to provide container vision without bloating your code or containers. Here is a docker compose overview that lays out the more in depth details from a service oriented point of view, even if your application's/service's containers are scattered in different Azure regions, availability sets, etc. Sysdig can provided a **unified holistic view** of the performance of that app/service. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure12.png" width="976" height="618" class="alignleft size-full wp-image-4062" alt="Swarm Monitoring" />][15] 



Sysdig is also getting DCOS data from the Orchestration API. Along with other application centric methods. you can see things such as Master and Agent node metrics, tasks, and even individual Marathon grouping 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure13.png" width="976" height="618" class="alignleft size-full wp-image-4062" alt="Swarm Monitoring" />][16] 



We can even provide granular application and or service level metrics for monitoring your Azure deployments. Here's an example node.js app running on the 3 agent docker swarm we deployed in our azure container service: 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure14.png" width="976" height="618" class="alignleft size-full wp-image-4062" alt="Swarm Monitoring" />][17] 



We are able to segment out which container is using x amount of heap size for instance…. All without anything installed in your containers or application code at all. 

### Service based access control to Azure container services… Can Sysdig do that? 

With [Sysdig Teams][18], we can isolate specific Azure instances to the end application teams needing visibility, we can even narrow it down to the end applications they are building. In this example I created two Teams, Azure-DCOS and AZURE-SWARM and created a service level access control to only visibility into the DC/OS Mesos environment 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure-17.png" width="976" height="618" class="alignleft size-full wp-image-4062" alt="Swarm Monitoring" />][19] 



after switching to the AZURE-DCOS team, we only see what metrics ive selected from the team creation. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure16.png" width="976" height="618" class="alignleft size-full wp-image-4062" alt="Swarm Monitoring" />][20] 

### What if I don’t want to use a SaaS monitoring solution? 

Sysdig can be installed our on premises installation in either a standard AZURE VM instance.



**Directions here:** <https://support.sysdigcloud.com/hc/en-us/articles/206519903-On-Premises-Installation-Guide> or your brand new Azure Container Services environment. (contact Sysdig for details) 

### Final thoughts 

Sysdig works extremely well with Azure and can you provide you and your team even more visibility into your applications in addition to the default Azure tools. We hope this blog will help you take the leap in testing your applications/containers with Sysdig and Azure. Sysdig is offering [14 day trial][21] if you want to take a test drive and you can also contact us if you want a demo.

 [1]: https://docs.microsoft.com/en-us/azure/container-service/container-service-deployment
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure1.png
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure2.png
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure3.png
 [5]: https://docs.microsoft.com/en-us/azure/container-service/container-service-monitoring-sysdig
 [6]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure4.png
 [7]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure5.png
 [8]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure6.png
 [9]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure7.png
 [10]: https://sysdigrp2rs.wpengine.com/product/integrations/
 [11]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure8.png
 [12]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure9.png
 [13]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure10.png
 [14]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure11.png
 [15]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure12.png
 [16]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure13.png
 [17]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure14.png
 [18]: https://sysdigrp2rs.wpengine.com/blog/introducing-sysdig-teams/
 [19]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure-17.png
 [20]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/azure16.png
 [21]: https://sysdigrp2rs.wpengine.com/sign-up/