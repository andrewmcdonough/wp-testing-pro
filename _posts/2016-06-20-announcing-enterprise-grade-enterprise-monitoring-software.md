---
ID: 2689
post_title: >
  Announcing enterprise-grade container
  monitoring software
author: Apurva Dave
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/announcing-enterprise-grade-enterprise-monitoring-software/
published: true
post_date: 2016-06-20 17:14:48
---
**TL;DR: You asked, we delivered. Now you can get Sysdig container monitoring technology as software that runs in your private cloud or virtual private cloud, in addition to our existing cloud service.** 
### “But I need it behind my firewall.”

Today at DockerCon 2016 [we announced the availability][1] of Sysdig’s enterprise-grade software offering to monitor and troubleshoot containerized applications. We have been working with our enterprise customers for quite some time on this offering, and are now pleased to make it generally available. 

The reason we decided to offer container monitoring software is simple: many of our customers run containers in their own data centers, and do not allow their application performance data to reside anywhere else. 

Why would container monitoring data not be permitted off site? We’ve come across three common reasons for that: 

1.  Security requirements: Many organizations implement strict security policies requiring all application infrastructure to remain confined within certain operational environments. We’ve worked with banking, insurance, media, and government organizations who have these requirements.
2.  Compliance requirements: Organizations that must comply with regulations such as PCI or HIPPA find it easier to maintain consistent procedures for managing and auditing data by operating all relevant software. This helps them avoid the “slippery slope” of deciding on a case-by-case basis which services can be in the cloud while others must remain behind the firewall. On the flip side, it would be onerous or simply impossible for most cloud services to comply with the specific policies of each organization they sell to.
3.  Cost of operations: Many companies operate at scale at which it’s efficient for them to operate their own hardware and software.



Now, just because you have a few more requirements than some other companies, it’s unlikely that you’re not using (or planning to use) containers in your infrastructure. In fact, many organizations are looking to containers as their next, internal platform-as-a-service infrastructure to further improve the efficiency of their physical resources. No need to worry though - Sysdig has your back. 

As I mentioned, we’re already running our software offering in production, so you don’t have to worry about using an untested offering. A little sampling of who’s using it: 

*   A Fortune 500 telecommunications equipment provider
*   An international biotechnology firm
*   A large government agency
*   One of the world’s largest media companies



### Containers for Container Monitoring



We think our approach to delivering our software solution has a number of advantages. Here’s a quick rundown of how it’s all put together: 

*   Feature parity with Sysdig Cloud. Our software offering is in fact the same back end that we operate in our multi-tenant cloud solution. That means [application autodiscovery][2], event correlation, [per-container resource metrics][3], and more. This makes your choice simpler: you do not have to decide what features you need, you just need to decide where it needs to run and who needs to run it. Typically new functionality comes to our software a few weeks after we release it to Sysdig Cloud. That gives us time to ensure it is operating as planned.
*   Same container instrumentation. Part of what makes [Sysdig really special is our unique instrumentation capabilities][4]. We let you see inside containers, without bloated per-container agents or insecure networking loopholes. Our software offering works with the exact same instrumentation - just deploy the sysdig container on each host you’d like to monitor and we’ll see it all.
*   (Optionally) scale-out deployment horizontally. For small environments or tests you can deploy the entire Sysdig back-end on a single host. In larger environments you can deploy across many nodes. As your infrastructure grows, you can continue to scale Sysdig’s back-end to meet your needs.
*   Proven data protection. Our underlying componentry - specifically Cassandra and MySQL, make it straightforward for you to backup your data.
*   The same great Sysdig engineering team supporting you. As one of our Fortune 500 customers recently told us, “We’re not used to getting fixes and feature requests resolved as fast as we do with Sysdig.” We’d love every customer to feel this way, and in order to do that we put all hands on deck when it comes to support.
*   Delivered in containers. It shouldn’t be surprising that, as the container-native monitoring company, we leverage containers to deliver our software offering to our customers. For our customers moving to containers, this means they can operate our software much like their other infrastructure. For those monitoring VMs or bare metal, it’s still really simple to get started.



### The other things you need to know



[Our software is licensed][5] annually using the same model as our cloud service - a per-host monitored model. Unlike our cloud service, there is no month-to-month option. We’ll work closely with you to set up the software - it’s relatively easy but every environment and every network is slightly different. Sometimes oddities do arise and we want to make sure we find them and solve them with you.



If you’re able, we suggest you use [Sysdig Cloud][6] as a test run for our core capabilities. That will give you the ability to trial our monitoring abilities without committing any physical resources. If not, no worries, we’ll go the software route with you.

 [1]: https://sysdigrp2rs.wpengine.com/press-releases/sysdig-announces-container-monitoring-software
 [2]: https://sysdigrp2rs.wpengine.com/blog/no-plugins-required-application-visibility-inside-containers/
 [3]: https://sysdigrp2rs.wpengine.com/blog/monitoring-greedy-containers-part-1/
 [4]: https://sysdigrp2rs.wpengine.com/blog/sysdig-vs-dtrace-vs-strace-a-technical-discussion/
 [5]: http://www.sysdigrp2rs.wpengine.com/pricing
 [6]: https://sysdigrp2rs.wpengine.com/sign-up/