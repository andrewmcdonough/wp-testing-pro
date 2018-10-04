---
ID: 2756
post_title: Sysdig Cloud summer 2016 release
author: Knox Anderson
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-cloud-summer-2016-release/
published: true
post_date: 2016-07-05 11:21:47
---
We’re really excited to round up all the great functionality we’ve released this past quarter on Sysdig Cloud into our Summer Release. Here we’re going to give you a quick run-down on the most important features and where possible link you to deeper information on how to use the features.

If you’ve got questions on any of these features, or want to assistance using them, we’d love to help you! Drop us a line at @sysdig or attend our [Summer Release Webinar on Wednesday, July 13th at 1:00 PM PDT][1] for in depth training on each new feature.

### Improved Out of the Box Experience

[**New Container Resource Utilization Metrics:**][2] We now support new metrics so you can actually see which containers are getting, say, 300% of allocated CPU and which ones are getting the short end of the stick. 

**[Mesos & DC/OS Integration:][3]** Sysdig and Sysdig Cloud now provide full metrics and metadata support for Apache Mesos, Mesosphere Marathon framework, and the Mesosphere Data Center Operating System (DC/OS). That means you can see deep into the applications running in your Mesos-based environment, and also use framework metadata to shift from monitoring physical resources (like hosts and containers) to monitoring logical resources (like apps and services).

**K8s host/port Autodiscovery:** We updated our agent to make our Kubernetes integration even easier! The Sysdig Cloud Agent will look for the process kube-apiserver or hyperkube with the --apiserver command line parameter. If found, the agent will automatically connect to the local Kubernetes API server to collect cluster data in addition to regular host metrics…. Meaning one command for full cluster visibility. 

**[Openshift Integration:][4]** This new integration brings the container monitoring goodness of Sysdig to Openshift users via an easy setup. Once deployed they can leverage metadata provided by Openshift to monitoring applications and infrastructure through a As we launch this, we already have multiple, large organizations leveraging a combination of OpenShift and Sysdig to deploy and monitor their internal Platforms-as-a-Service operations.

### Advanced Alerting

**[Multi-Condition Alerting:][5]** Fine tune your alerts with new functionality. Advanced alerts allow you to define a threshold as a custom Boolean expression which can involve multiple conditions. These advanced expressions require a specific syntax, which we dive into further in this post.

**[Slack Alert Notifications:][6]** We love Slack and I know many of you do too. So now you can monitor containers (and your apps!) without ever leaving Slack, using our really easy-to-use integration for alerts from Sysdig Cloud.

**Custom Events Support:** Sysdig Cloud can ingest any custom event you want, including code deploys, auto-scaling activities, business level actions, etc. Sysdig Cloud will automatically overlay these events on charts and graphs, for easy correlation with your performance data. 

### New Tools from Sysdig

**[Sysdig Enterprise on Premise:][7]** At DockerCon 2016 we announced the availability of Sysdig enterprise-grade software - in addition to our cloud service - to monitor and troubleshoot containerized applications. We have been working with our enterprise customers for quite some time on this offering, and are now pleased to make it generally available.

The reason we decided to offer container monitoring software is simple: many of our customers run containers in their own data centers, and do not allow their application performance data to reside anywhere else.

**[Sysdig Falco:][8]** We released sysdig falco, a behavioral activity monitoring agent that is open source and comes with native support for containers. Falco lets you define highly granular rules to check for activities involving file and network activity, process execution, IPC, and much more, using a [flexible syntax][9]. Falco will notify you when these rules are violated. See Falco in action this week in our webinar - [Introducing Sysdig Falco: The Behavioral Activity Monitor With Container Support][10]

 [1]: https://zoom.us/webinar/register/b9960575ef8b64c966858a512be5123a
 [2]: https://sysdigrp2rs.wpengine.com/blog/monitoring-greedy-containers-part-1/
 [3]: https://sysdigrp2rs.wpengine.com/blog/monitoring-mesos/
 [4]: https://blog.openshift.com/openshift-ecosystem-using-sysdig-monitor-openshift/
 [5]: https://sysdigrp2rs.wpengine.com/blog/multi-condition-alerting/
 [6]: https://sysdigrp2rs.wpengine.com/blog/container-alerting-via-slack/
 [7]: https://sysdigrp2rs.wpengine.com/blog/announcing-enterprise-grade-enterprise-monitoring-software/
 [8]: http://www.sysdig.org/falco/
 [9]: https://github.com/draios/falco/blob/dev/rules/falco_rules.yaml
 [10]: https://zoom.us/webinar/register/b3aba0e4f3317898dc2040ba88984b7b