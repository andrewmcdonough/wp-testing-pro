---
ID: 7785
post_title: >
  Amazon EKS monitoring and security with
  Sysdig.
author: Eric Carter
post_excerpt: >
  Amazon Elastic Container Service for
  Kubernetes (Amazon EKS) provides
  Kubernetes as a managed service on AWS.
  It helps make it easier to deploy,
  manage and scale containerized
  applications on Kubernetes.
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/amazon-eks-monitoring-and-security-with-sysdig/
published: true
post_date: 2018-07-17 04:26:24
---
Amazon Elastic Container Service for Kubernetes ([Amazon EKS][1]) provides Kubernetes as a managed service on AWS. It helps make it easier to deploy, manage and scale containerized applications on [Kubernetes][2]. Sysdig cloud-native intelligence solutions – [Sysdig Monitor][3], and [Sysdig Secure][4] – provide Amazon EKS monitoring and security from a single agent and unified platform. Sysdig helps AWS customers see more, secure more, and save time in troubleshooting deployed microservices. 
## Container and orchestration insight for Amazon EKS {#containerandorchestrationinsightforamazoneks} Why Sysdig is so effective for monitoring and securing Amazon EKS and any Kubernetes environment is our approach to container and orchestration integration. ContainerVision™ gives you request‐level visibility inside your containers and across microservices, providing the in-depth metrics and events without invasive instrumentation. ServiceVision™ integrates with Kubernetes to automatically enrich all your metrics and events with orchestration metadata. Together these technologies help you pinpoint where in your cluster you are experiencing a performance problem or security event. With the ability to visualize and segment information by Kubernetes logical resources, like namespace, deployment, or pod, you can see exactly what services are being impacted, and where. Your Amazon EKS environment will include thousands of 

[labels and tags][5] exposed by your infrastructure, containers, microservices, etc. Sysdig automatically collects these labels and tags and lets you group and segment your information, making it possible to "slice and dice" your environment views by physical (e.g. EC2 instances) and logical (Kubernetes nodes, pods, etc.) details to see what you need to see in a rich and powerful way. For Amazon EKS monitoring and security, this means you have at your fingertips in-depth views from top level dashboards to individual metrics and security-event views, all the way down the process level. And when something happens, say a pod crash and restart loop, or a data exfiltration event, you’ll be able to dig in to the details to find the needle in the haystack and fix the problem. [tweet_box design="default" float="none"]Moving to Amazon Elastic #Container Service for #Kubernetes? Get unified #monitoring and #security for #EKS with #Sysdig[/tweet_box] 
## Getting started with Sysdig and Amazon EKS {#gettingstartedwithsysdigandamazoneks} Getting started with Sysdig on Amazon EKS is simple and straightforward. It requires a simple container-agent installation, shipped as a Docker container and deployed with a DaemonSet. The DaemonSet installation with Kubernetes ensures that all Nodes run a Pod with Sysdig and automatically adds the monitoring and security agent as nodes are added to your cluster. 

[See how here.][6] As you deploy services in your environment, Sysdig monitors [system calls at the kernel level][7] to automatically collect deep information from your AWS instances, containers and EKS for real-time monitoring and security including: 
*   [Performance and health metrics][8]
*   [Kube-state metrics][9]
*   Custom metrics like [Prometheus][10], [JMX][11] and [StatsD][12]
*   [System and security events][13]
*   [System captures][14]

## 360-degree views of Amazon EKS with Sysdig {#ohthethingsyoucandowithsysdigandamazoneks} Once you’re up and running, with Sysdig there are many ways to explore, view and analyze your Amazon EKS environment. 

*   **Spotlight:** [Sysdig Spotlight][15] gives you a quick overview of what’s been discovered in your environment, the integrations that are running (or not), and status of your running agents. It’s job is to help you maintain a healthy environment and see at-a-glance how your services on EKS are doing.

[<img class="alignnone size-full wp-image-7789" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/EKS-spotlight.png" alt="EKS Sysdig Spotlight" width="1440" height="900" />][16]   
*   **Explore:** Sysdig enables you to explore your Amazon EKS environment to see what’s happening at any level. You can view hosts and containers, or apply a Kubernetes grouping, drilling into you environment instead by something like AWS region > cluster > namespace > deployment > pods > containers. <span style="font-weight: 400;">This provides an HTOP-like view of metrics like CPU, disk, memory, and network from the entire infrastructure down to the container-level. Color coding helps you spot potential issues quickly. From here you can drill into dashboards and metric views automatically scoped by your selection in the explore tree.</span>

[<img class="alignnone size-full wp-image-7790" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/EKS-explore.png" alt="EKS Sysdig Explore" width="1440" height="900" />][17]   
*   **Topology maps:** Automatically, Sysdig will create for you topology maps that you can use to view how your services, containers, and processes are communicating, view response times and latency, check network traffic and view CPU utilization. With EKS you’ll automatically see your resources by region, availability zone and cluster and be able to drill down to see fine-grained details.

[<img class="alignnone size-full wp-image-7791" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/EKS-topology.png" alt="EKS Sysdig Topology" width="1440" height="900" />][18]   
*   **Dashboards:** With Dashboards, you can view a summary of things like pod health with kube-state-metrics, or Kubernetes service health with Golden Signals. These dashboards (and more) are included "out of the box," but you can also build your own custom views for any of the EKS metrics or information that is most important to you.

[<img class="alignnone size-full wp-image-7793" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/EKS-dashboards.png" alt="EKS Sysdig dashboard" width="1440" height="900" />][19]   
*   **Alerts:** Adaptive alerts let you get notified automatically via email, PagerDuty, Slack, etc. when events occur like exceeding a metric threshold, or with any violation occurring the violates your set security policies. Kubernetes node out of disk? Deployments degraded? Pods crashing? Someone running an unauthorized program in a container? You can get get alerted immediately.

[<img class="alignnone size-full wp-image-7792" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/EKS-alerts.png" alt="EKS Sysdig alerts" width="1440" height="900" />][20]   
*   **Security events:** For your security team, you can get a summary of events for the last hour, or the last week, etc. and drill into policy violations in your EKS deployment. For example, if an an attempt to read sensitive files (e.g. files containing user/password/authentication information) take place, you’ll be able to identify, block, and further investigate the issue.

[<img class="alignnone size-full wp-image-7794" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/EKS-security-event.png" alt="EKS Sysdig security event" width="1440" height="900" />][21]   
*   **Captures:** Security forensics and Kubernetes troubleshooting are simplified with Sysdig captures, letting you analyze system call and environment data, and correlate details surrounding any alert or event using integrated open source [Sysdig Inspect][22]. Sysdig gives you the information you need even after containers are killed or gone.

[<img class="alignnone size-full wp-image-7795" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/EKS-troubleshooting-forensics.png" alt="EKS Sysdig forensics troubleshooting" width="1440" height="900" />][23] 
## Learn more about Sysdig and Kubernetes {#learnmoreaboutsysdigandkubernetes} At Sysdig we’ve invested a lot of effort into providing feature-rich support Kubernetes - and now Amazon EKS. Our goal is to provide you with the intelligence you need to simplify your job of ensuring the containerized services you run on Amazon EKS are secure and performing at their best. To learn more about what some of the largest companies in the world are doing with Docker, Kubernetes, Amazon, Sysdig Monitor and Sysdig Secure - download and read 

[6 Real-World Kubernetes Use Cases][22] and check out our related resources below.

 [1]: https://aws.amazon.com/eks/
 [2]: https://kubernetes.io/
 [3]: https://sysdigrp2rs.wpengine.com/product/monitor/
 [4]: https://sysdigrp2rs.wpengine.com/product/secure/
 [5]: https://sysdigrp2rs.wpengine.com/blog/understanding-container-metadata/
 [6]: https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/206770633
 [7]: https://sysdigrp2rs.wpengine.com/blog/fascinating-world-linux-system-calls/
 [8]: https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/204931155
 [9]: https://sysdigrp2rs.wpengine.com/blog/introducing-kube-state-metrics/
 [10]: https://sysdigrp2rs.wpengine.com/blog/prometheus-metrics/
 [11]: https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/204178959
 [12]: https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/204376099
 [13]: https://sysdigrp2rs.wpengine.com/blog/sysdig-secure-docker-run-time-security/
 [14]: https://sysdigrp2rs.wpengine.com/blog/new-integrated-troubleshooting-sysdig-monitor/
 [15]: https://sysdigrp2rs.wpengine.com/blog/introducing-sysdig-spotlight-new-toolkit-discovery-maintenance/
 [16]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/EKS-spotlight.png
 [17]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/EKS-explore.png
 [18]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/EKS-topology.png
 [19]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/EKS-dashboards.png
 [20]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/EKS-alerts.png
 [21]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/EKS-security-event.png
 [22]: https://go.sysdigrp2rs.wpengine.com/l/231542/2018-07-03/c89rv
 [23]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/EKS-troubleshooting-forensics.png