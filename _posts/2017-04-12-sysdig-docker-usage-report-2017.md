---
ID: 3977
post_title: The 2017 Docker Usage Report
author: Apurva Dave
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-docker-usage-report-2017/
published: true
post_date: 2017-04-12 06:00:50
---
Many Docker usage reports are based on user surveys. These are incredibly useful to understand intent and objectives, but can sometimes be inconsistent with how people are actually adopting Docker within their environments right now. The main question we wanted to answer was, “How are people using Docker in their application environments *right now?*” As the premier container monitoring solution, Sysdig is in a fantastic position to answer this question with actual Docker usage data across hundreds of customers.   
  
Our high level answer on Docker usage: It’s growing, both in scale but also sophistication. Read on to find out more.  
  
[tweet_box design="default" float="none"]We analyzed usage of 45,000 #Docker containers through #monitoring data. Here's what we found.[/tweet_box]  
The data you see here represents a snapshot of our customer behavior in early spring 2017. We have aggregated the data across all environments to provide unique insight into how our customers are using Docker containers right now. **This is not meant to be a broad statement on the market as a whole - we can only speak to what we can report on from our own data**.   
## Sample size of the Docker usage survey  
  
[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/image05-1024x245.jpg" alt="Docker usage report sample size" class="aligncenter size-large wp-image-3984" height="245" width="1024" />][1] There are lots of containers out there. A lot. We couldn't pull data across all customers + all hosts + all containers for the purpose of this report. So instead, we took a point-in-time snapshot of a representative sub-sample of the containers we monitor via our cloud service for the purpose of this report. The sample numbered 45,000 containers. We also provide an on-premises container monitoring solution, but that data is not included within this Docker usage report. 

  
## Container density as a measure of adoption  
  
[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/image01-1024x173.jpg" alt="Container Density to measure docker usage" class="aligncenter size-large wp-image-3980" height="173" width="1024" />][2] One of the initial advantages that many teams see in using containers is their ability to provide “lightweight virtualization” as compared to hypervisors. The lightweight nature implies that they can run at a high container density, make better use of underlying physical resources, and thereby save money.

  
We wanted to see if users were actually seeing this benefit. We asked the question, "Of customers running containers in their environments, what is the median number of containers being run?” At a median of ten containers per host, it seems as though Docker users are beginning to see some benefit in density. Note, deriving this median was a two step process: this median is derived by first calculating the average number of containers per host across each customer, and then calculating the median across customers. This felt like a more accurate way to understand where customers are with regards to density. We also saw a very broad range of deployment densities.   
  
**We saw some customers running as high as 95 containers per host,** and others who are running at a single container per host. The latter situation, while it may sound unusual, has underlying logic. Talking to those customers, we have learned that their core software uses all the resources on the machine. The benefit for containerization to them is *not* providing container density. Rather, it’s the ability to develop, deploy, and scale software more quickly where Docker adds value.   
## What orchestrators do people use with Docker containers?[  
[tweet_dis_img]<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/image02-1024x274.jpg" alt="Orchestrator related to docker usage" class="size-large wp-image-3981 alignnone" height="274" width="1024" />[/tweet_dis_img]][3] The popularity of Kubernetes is not surprising. In this stage of Docker adoption we saw Kubernetes take an early lead against other orchestrators, and 

[monitoring Kubernetes][4] has been a hot topic around the Sysdig office. However, with the newer version of Swarm, and the increasing efforts behind Open DC/OS, we have seen an increased variety of orchestrator platforms in use. These platforms have not caught up to Kubernetes within our customer base yet, but we expect to see continued competition here as these platforms mature. The results above also pull in enterprise-class distributions of the base orchestrator.   
  
For example:   
*   Kubernetes includes Openshift, Tectonic and others
*   Mesos includes Mesosphere DC/OS
*   Swarm includes both the older version and native Swarm The other observation here is the amount of users that fall into the “Other” category. These are a mix of customers who use no orchestration (likely to be very small deployments), built their own custom orchestration, or just use Docker labels to manage everything manually. We have seen companies who fall into the “other category” decrease significantly over time, and expect that trend to continue. 

  
## Most popular container registries

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/image04-1024x376.jpg" alt="what container registries are in use with Docker containers?" class="aligncenter size-large wp-image-3983" height="376" width="1024" />][5]  
  
A container registry is a place to store and distribute Docker images. You can think of a registry as a gas station - no matter where you go the gas is really the same, but one station might have decent coffee, another throws in a free car wash, and yet another is attached to the Safeway so you can get your groceries and gas at once.   
  
Anyone who is seriously using Docker containers will likely be using a registry, either as a service or as on premise software (aka a private registry.) As you can see from the numbers above, even the top three combined don’t constitute a majority of the Docker containers in use within the sample. That’s indicative of how fragmented the registry offerings are. There are two footnotes to this data: (1) these percentages are based on container count, not based on registry per customer; (2) For a portion of our sample, the data did not allow us to accurately identify which container registry is in use. We assume that these are evenly distributed - nothing to our knowledge would indicate otherwise.   
## The top 12 application components running in containers  
  
[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/image08-1024x377.jpg" alt="what applications do people use inside Docker containers?" class="aligncenter size-large wp-image-3987" height="377" width="1024" />][6] What are people actually running inside their containers? Sysdig’s 

[auto-discovery mechanism][7] gives us a method to understand which open source components users are adopting inside their containers, without any explicit configuration or management by users. Above is a list of the twelve most common application checks we see running.   
  
[tweet_box design="default" float="none"]#Nginx, #etcd, & #Consul top the list of components running inside #Docker #containers[/tweet_box]  
Of interest to us was that Nginx was the most popular. It is frequently serving at the endpoint of a service, or functioning as a traditional load balancer. Also somewhat surprising to us was that consul was running so closely to etcd - we expected more distance between the two. Note this data does not focus on programming languages: most users are writing at least some custom application code in various languages and we see those as “custom apps” if a customer chooses to write their own application check.   
## Popular alert conditions used with Docker containers

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/image03-1024x431.jpg" alt="Popular Alert Conditions Used with Docker Containers" class="aligncenter size-large wp-image-3982" height="431" width="1024" />][8]  
  
We wanted to better understand if docker adoption had changed the way people architect and operate their software. While alerts are not a perfect way to understand such a deep question, they do provide a glimpse into how people are thinking about their software in production. Most importantly, this data shows us a shift away from host/physical infrastructure alerting is in process, but older alerting conditions are still in use.  
The image above shows you the most popular alert conditions, though they are not ranked. We felt that, given alert conditions really need to be paired with scope (see next section), it made the most sense to leave off the actual count of conditions used.    
  
For example, tried-and-true alerts such as High CPU/Memory/Disk are still very much in use. At the same there are newer alert conditions which better relate to today’s abstracted infrastructure. A good example here is “Pod Restart Count”. We also see a number of alerts that adapted for the modern age: “Entity is Down” now is used across containers in addition to hosts, and “High CPU shares” is a CPU alert tuned for containers.   
  
Finally, we see a number of alerts that haven’t changed nor should they. Alerts on the response time of a service and alerts on HTTP Error Codes continue to be popular because they represent the state of the application, not just the state of container infrastructure. The biggest change with these alerts tends to be the scope across which they trigger, which we cover next.   
## Popular alert scopes used with Docker containers  
  
[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/image06-1024x545.jpg" alt="Popular Alert Scopes Used with Docker Containers" class="aligncenter size-large wp-image-3985" height="545" width="1024" />][9] We see a few things when examining alert scopes in relation to how customers use Docker containers. First is the concept of tagging. This allows DevOps teams to make non-hierarchical groups of containers that serve particular purposes. Examples include an environment (dev/prod/test), a physical location (cloud provide zone or data center), or micro-service name. 

  
  
The most common number of tags we see used to scope out an alert are 2 to 3 tags. Many of the alert scopes are actually operating across orchestrator constructs, such as a Kubernetes Deployment, Pod, or something similar. We also see, however, that many people are using the container name, which implies the user has a more personal understanding of what is running where inside their infrastructure.   
  
Finally, we do see some important tagging related to physical infrastructure as well. Cloud provider tags typically represent some physical aspect of deployment, such as the host, region or availability zone. The “Role of the Host” is typically applied at the Sysdig agent. Given users only need to deploy a single sysdig agent per host - regardless of the number of containers or pods - this implies they are using the tag in relation to physical infrastructure.   
## Conclusion: Docker usage continues to advance Our experience in the Docker ecosystem is that users continue to advance in both the scale and sophistication of their Docker usage. This report gives the reader a good sense of where users are today, and provides a complement to survey-based reports that provide more insight on intention or objectives. We expect to repeat this report in 

[about a year][10], and imagine that we will see significant changes in that time.  
  
**Would you like a downloadable PDF version of this report?** [Grab it here][10].  
#### *The 2018 report is now available! Access it today by [clicking here][10].*

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/image05.jpg
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/image01.jpg
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/image02.jpg
 [4]: https://sysdigrp2rs.wpengine.com/blog/monitoring-kubernetes-with-sysdig-cloud/
 [5]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/image04.jpg
 [6]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/image08.jpg
 [7]: https://sysdigrp2rs.wpengine.com/blog/no-plugins-required-application-visibility-inside-containers/
 [8]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/image03.jpg
 [9]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/image06.jpg
 [10]: https://go.sysdigrp2rs.wpengine.com/docker-usage-report