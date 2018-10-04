---
ID: 5912
post_title: 'Sysdig&#8217;s Top Viral Blogs of 2017'
author: Knox Anderson
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdigs-top-viral-blogs-2017/
published: true
post_date: 2018-01-18 12:01:16
---
2017 was a big year for Sysdig: <a href="https://github.com/draios/sysdig-inspect" target="_blank">New opensource projects</a>, a <a href="https://sysdigrp2rs.wpengine.com/press-releases/sysdig-raises-25m-for-the-container-intelligence-platform/" target="_blank">round of funding</a>, <a href="http://www.sysdigccwfs.com/" target="_blank">community conferences</a>, and even a new product, <a href="https://sysdigrp2rs.wpengine.com/product/secure" target="_blank">Sysdig Secure</a>, to set the foundation for what will be a great 2018. While all of that is awesome, what really brings you to our blog is the content that dives deep into system internals. 

Let’s recap 2017 by focusing on the viral blogs that climbed up hackernews and into your twitter feeds. We’re going by the data and looking at the blogs that you, the readers visited most in 2017. 

### [1\. Container isolation gone wrong][1] {#1containerisolationgonewrong}

At it’s peak this blog had 164 users coming to the blog per minute. Containers can have separate quotas for CPU, memory, storage… what could possibly go wrong? 

One of the main advantages of embracing containers is "lightweight virtualization". Since each container is just a thin layer around the containerized processes, The user gains enormous efficiencies, for example by increasing the container density per host, or by spinning containers up and down at a very fast pace.

However, as the troubleshooting story in the article will show, this lightweight virtualization comes at the cost of sharing the underlying kernel among all containers, and in some circumstances this can lead to surprising and undesirable effects that container users typically don’t think about.

This troubleshooting tale is rather involved. I’ve started from the basics and worked up to the more complex material in the hope that readers at all levels can get value out of it. [Read More][1]

### [2\. Kubernetes monitoring guide][2] {#2kubernetesmonitoringguidepart1}

If 2016 was the year of containers, then 2017 was the year of Kubernetes. This blog kicks off a 4 part series covering Kubernetes monitoring, alerting, troubleshooting, and how customers are actually using kubernetes in production. 

With over 21,000 stars on GitHub, over a thousand committers, and an ecosystem that includes Google, RedHat, Intel, and more, Kubernetes has taken the container ecosystem by storm. It’s with good reason too: Kubernetes acts as the brain for your distributed container deployment. It’s designed to manage service-oriented applications using containers distributed across clusters of hosts. Kubernetes provides mechanisms for application deployment, service discovery, scheduling, updating, maintenance, and scaling. But what about monitoring Kubernetes? [Read More][2]

### [3\. Docker usage report][3] {#3dockerusagereport}

As someone who’s attended DockerCon for the last 3 years it’s been pretty impressive to see the community grow around containers. This data driven post looks at how people are using Docker in their application environments right now. 

Many Docker usage reports are based on user surveys. These are incredibly useful to understand intent and objectives, but can sometimes be inconsistent with how people are actually adopting Docker within their environments right now. The main question we wanted to answer was, "How are people using Docker in their application environments *right now?*" As the premier container monitoring solution, Sysdig is in a fantastic position to answer this question with actual Docker usage data across hundreds of customers. [Read More][3] [tweet_box design="default" float="none"]The top 5 #sysdig viral container blogs of 2017[/tweet_box] 

### [4\. Sysdig Inspect][4] {#4sysdiginspect}

Ok, Ok, I know earlier I said we’d keep release posts out of the list, but Sysdig Inspect was too good. It’s not everyday that someone builds an Opensource GUI for system call analysis.

Sysdig is routinely praised by users for the richness of the data it’s able to capture and for the ability to store system, application and container that data into capture files that can be easily shared with other people.

However, extracting insights from rich data can be hard. Mastering the art of analyzing sysdig capture files requires dedication and skills. This is why we constantly try to improve workflows around sysdig and find ways to get better insights with less effort. Today, we bring these efforts to a new level with the release of Sysdig Inspect.

Sysdig Inspect is a powerful, intuitive tool for sysdig capture analysis that runs natively on your Mac or your Linux PC, with a user interface that has been designed for performance and security investigation. [Read More][4]

<div style="text-align:center, clear:both">
  <iframe width="560" height="315" src="https://www.youtube.com/embed/M1W8txpJKxY" frameborder="0" allowfullscreen></iframe>
</div>

### [5a. Top 20 Docker security tools][5] {#5atop20dockersecuritytools}

As more and more companies move into production environments with containers, security becomes ever more important. This post rounds up the container security ecosystem through every layer of your security stack, from scanning down to forensics. We are keeping the number "20" in the title, but the list has 22 items at this moment… and growing.

There are quite a few Docker security tools in the ecosystem, how do they compare? This is a comprehensive list of Docker security tools that can help you implement some of the container security best practices.

**Is Docker insecure?** Not at all. Actually features like process isolation with user namespaces, resource encapsulation with cgroups, immutable images and shipping the minimal software and dependencies reduce the attack vector providing a great deal of protection. But, is there anything else we can do? There is much more than image vulnerability scanning and these are 20 container and Docker specific security tools that can help. [Read More][5]

### [5b. Docker vulnerabilities and threats][6] {#5bdockervulnerabilitiesandthreats}

A list of container vulnerabilities is great, but it’s much more useful if it includes proof of concept examples how to address them in your environment. This post has been one of the more consistently visited pages since it’s release. 

There are some specific parts of Docker based architectures which are more prone to attacks. In this article we are going to cover **7** fundamental Docker security vulnerabilities and threats.

Each section has been divided into:

*   Threat description: Attack vector and why it affects containers in particular.

*   Docker security best practices: What can you do to prevent this kind of security threats.

*   Proof of Concept Example(s): A simple but easily reproducible exercise to get some first hand practice.

### Docker vulnerabilities and threats to battle {#dockervulnerabilitiesandthreatstobattle}

*   <a href="https://sysdigrp2rs.wpengine.com/blog/7-docker-security-vulnerabilities/#docker-host-and-kernel-security" target="_blank">Docker host and kernel security</a>
*   <a href="https://sysdigrp2rs.wpengine.com/blog/7-docker-security-vulnerabilities/#docker-container-breakout" target="_blank">Docker container breakout</a>
*   <a href="https://sysdigrp2rs.wpengine.com/blog/7-docker-security-vulnerabilities/#container-image-authenticity" target="_blank">Container image authenticity</a>
*   <a href="https://sysdigrp2rs.wpengine.com/blog/7-docker-security-vulnerabilities/#container-resource-abuse" target="_blank">Container resource abuse</a>
*   <a href="https://sysdigrp2rs.wpengine.com/blog/7-docker-security-vulnerabilities/#docker-security-vulnerabilities-present-in-the-static-image" target="_blank">Docker security vulnerabilities present in the static image</a>
*   <a href="https://sysdigrp2rs.wpengine.com/blog/7-docker-security-vulnerabilities/#docker-credentials-and-secrets" target="_blank">Docker credentials and secrets</a>
*   <a href="https://sysdigrp2rs.wpengine.com/blog/7-docker-security-vulnerabilities/#docker-runtime-security-monitoring" target="_blank">Docker runtime security monitoring</a>

## Bonus content

We’re stacking the deck in 2018 and have already released two great posts to start of the new year. 

### [Making sense of Meltdown/Spectre][7] {#makingsenseofmeltdownspectre}

The IT world was put on its heels this past week as two of the most significant hardware exploits were announced for CPUs from Intel, ARM, AMD, IBM, and others. Early estimates state that as many as 3 billion devices have this flaw (don’t worry, <a href="https://twitter.com/MarkKriegsman/status/949355793941491718" target="_blank">your Apple II and C64 is safe</a>). As there often is with a vulnerability of this magnitude, there’s a ton of noise associated with the problem, which makes it difficult for the average IT professional to get to the facts. So let’s step back from the hyperbole, and break down exactly what the problem is, and how it affects the day to day life of the average IT professional.

### [Fishing for miners – cryptojacking honeypots in Kubernetes][8] {#fishingforminerscryptojackinghoneypotsinkubernetes}

We run Kubernetes honeypots so you don’t have to. What would happen if you put a K8s cluster up on AWS but left the API Server port (8080) generally open to the outside world? Have attackers moved on from EC2 exploits to container-specific and kubernetes-specific exploits? One way to find out!

 [1]: https://sysdigrp2rs.wpengine.com/blog/container-isolation-gone-wrong/
 [2]: https://sysdigrp2rs.wpengine.com/blog/monitoring-kubernetes-with-sysdig-cloud/
 [3]: https://sysdigrp2rs.wpengine.com/blog/sysdig-docker-usage-report-2017/
 [4]: https://sysdigrp2rs.wpengine.com/blog/sysdig-inspect/
 [5]: https://sysdigrp2rs.wpengine.com/blog/20-docker-security-tools/
 [6]: https://sysdigrp2rs.wpengine.com/blog/7-docker-security-vulnerabilities/
 [7]: https://sysdigrp2rs.wpengine.com/blog/making-sense-of-meltdown/
 [8]: https://sysdigrp2rs.wpengine.com/blog/detecting-cryptojacking/