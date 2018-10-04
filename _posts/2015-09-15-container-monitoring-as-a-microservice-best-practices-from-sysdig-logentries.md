---
ID: 2052
post_title: 'Container Monitoring as a Microservice: Best Practices From Sysdig &#038; Logentries'
author: Marcus Sarmento
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/container-monitoring-as-a-microservice-best-practices-from-sysdig-logentries/
published: true
post_date: 2015-09-15 07:00:56
---
I recently had a chance to speak with Trevor Parsons, Co-Founder of Logentries, ahead of our joint webinar where we'll be discussing <a href="http://info.logentries.com/container-monitoring-as-a-microservice?le_tofu=webinar-sysdig-email" target="_blank">Container Monitoring as a Microservice</a>. I had an opportunity to ask him about his thoughts on the state of the container market and am excited to share his perspective with you. Remember to <a href="http://info.logentries.com/container-monitoring-as-a-microservice?le_tofu=webinar-sysdig-email" target="_blank">register for the webinar</a> to hear Trevor and Chris Crane, COO of Sysdig, discuss what types of container metrics you should be monitoring and measuring, state of the art container visibility techniques, and the pros and cons of each. 

**Marcus:** What's going with containers? 

**Trevor:** We’ve noticed in the last 6 months or so that there has been a significant shift in containers being used for real production workloads. We are seeing more and more of our users building their systems using Docker containers for example. The characteristics of these systems are very often highly dynamic, where a system may autoscale to 100’s or 1000’s of container instances very quickly. Container instances are also often ephemeral in nature, possibly only existing for a short amount of time. 

**Marcus:** What types of challenges do containers introduce? 

**Trevor:** Most of the common challenges I think people are facing with containers aren’t necessarily new, but need to be reconsidered given containers’ disparate, ephemeral nature. These challenges include networking, operational logistics like storage, security and monitoring. Monitoring has been particularly challenging to users looking to understand what is happening across these (often very) dynamic environments. However there have been some recent developments (e.g. APIs, new monitoring approaches, logging support) that have helped to make this a lot easier. 

In our <a href="http://info.logentries.com/container-monitoring-as-a-microservice?le_tofu=webinar-sysdig-email" target="_blank">upcoming webinar</a>, COO of Sysdig Chris Crane and I will be exploring how Sysdig and Logentries are helping to tackle this challenge. 

**Marcus:** What are some examples of container metrics you think are important to monitor and measure? 

**Trevor:** To get a comprehensive view of what is happening across your environment you will need to collect metrics at a number of different levels e.g. application level metrics, container stats, Docker API events, system level resource usage, etc. 

The type of data you need to collect obviously also depends on the questions you are looking to answer as you monitor your system. For example, if you want to track latency across a transaction that spans different micro-services, you may want to log unique identifiers (e.g. request ID) along with the request latency such that you can track a request and where most time is spent across the different micro-services that service that request. 

Chris and I will be demonstrating how to collect and correlate metrics from the different levels of your container stack using a combination of Sysdig Cloud and Logentries during our <a href="http://info.logentries.com/container-monitoring-as-a-microservice?le_tofu=webinar-sysdig-email" target="_blank">upcoming webinar</a>. 

**Title:** Container Monitoring as a Microservice 

**Date:** Thursday, September 17, 2015 

**Time:** 11am PDT / 2pm EDT 

<a href="http://info.logentries.com/container-monitoring-as-a-microservice?le_tofu=webinar-sysdig-email" target="_blank">Register here to reserve your seat!</a>