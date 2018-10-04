---
ID: 6462
post_title: >
  Sysdig announces expanded Prometheus
  support options
author: Apurva Dave
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/prometheus-support/
published: true
post_date: 2018-02-27 03:00:28
---
Today we're excited to launch new options to support Prometheus monitoring users everywhere.

Whether you're running open source Prometheus yourself or looking for better Prometheus integration in our [Sysdig Monitor product][1], we've got some new stuff for you.

Here's what we're talking about today: 
1.  We are declaring our love (again) for Prometheus. It's like Valentine's day all over again.
2.  We're announcing new support subscriptions for your self hosted, open source Prometheus deployment. This is a completely new offering, complimentary to our commercial products.
3.  You may not have known this, but we already support Prometheus metrics within our commercial product Sysdig Monitor. But that was just the beginning, we're continuing to develop more integrations, so we'll give you a sneak preview into what's coming next in our roadmap.

## Sysdig’s philosophy on Prometheus {#sysdigsphilosophyonprometheussupport}

Overall, our philosophy is to be o*pen by design* in the quest to build our container intelligence platform. To that end, we’re big fans of Prometheus. The developer community at large uses Prometheus for Kubernetes monitoring, so we wanted to ensure we have a seamless support.

We started <a href="https://sysdigrp2rs.wpengine.com/blog/prometheus-metrics/" target="_blank" rel="noopener">officially supporting Prometheus monitoring</a> last year, when we integrated metric support alongside StatsD and JMX. And we continue our work to make Prometheus a first class citizen in Sysdig Monitor (roadmap spoilers ahead, keep reading!).

But we saw that many early stage users were still just using vanilla, open source Prometheus, self-hosted in their own environment.

[][2]*Sysdig already supports Prometheus metrics within [Sysdig Monitor][1]*

And we have decided to help those users too!

## New support subscription packages for self-hosted Prometheus {#newsupportpackagesforselfhostedprometheus}

We’ve found that many enterprises who are just beginning their Kubernetes journey start by setting up vanilla, open source Prometheus to provide early performance measurement and monitoring of their test environment. In many cases, this Prometheus environment transitions into production with the new software.

<img src="/wp-content/uploads/2018/02/Prometheus-metrics-Sysdig-Monitor-UI.png" alt="Prometheus Support in sysdig monitor" width="1200" height="750" class="alignleft wp-image-4350" />



It's usually right before (or, ahem, right after) software moves into production that teams get serious about reassessing their operational tooling. Does it fit their needs? Is it scalable & reliable enough? How much time goes into management of the tools? Have the management features the DevOps team requires? In the meantime, keeping what they’ve got up and running is essential.

That's why today we're announcing [support subscriptions][3] for your self-hosted Prometheus environment. If you're already up and running with Prometheus, now you can leverage Sysdig support as a backstop for questions, configuration issues, performance concerns, or issues you're having.  
  
We know that you’ve got software to run, and your life shouldn’t be tied up in running your monitoring software behind the scenes. Support packages start at $15 per host per month, but can be tuned to your environment. Containers monitored, scale of metrics, and response times can be customized to what you need (pricing may vary based on this stuff).<tweet></tweet>[tweet_box design="default" float="none"]<tweet>@Sysdig now offers support for your self-running #Prometheus deployment for #Kubernetes monitoring</tweet> 

##  {#aroadmapforfurtherprometheussupportinsysdigmonitor}[/tweet_box]

## A roadmap for further Prometheus integration in Sysdig Monitor {#aroadmapforfurtherprometheussupportinsysdigmonitor}

While we believe our new Prometheus support packages will serve a lot of customers well, Sysdig already has hundreds of customers who use Sysdig Monitor as a fully supported commercial software product, in order to reduce their internal management time and resources and implement enterprise-class functionality. Incorporating Prometheus support alongside a range of our existing features is a natural next step for us.

In particular, for customers who were previously using Prometheus, or using Prometheus alongside Sysdig, here are a couple of the key areas they wanted us to address:

*   "Some of our application development teams are already using Prometheus. How do I make sure they have all the functionality they need, without having to manage a backend?"

*   "We like using Grafana as our front end. We’ve already built a lot of dashboards. How can Sysdig leverage this?"

*   "We use Prometheus, but our tests show we can’t scale a single deployment to the size or data retention we need. What should we do?"

[<img style="max-width: 100%; height: auto;" src="/wp-content/uploads/2018/02/Prometheus-metrics-Grafana.png" alt="Prometheus metrics grafana" width="1200" height="750" class="alignleft wp-image-4350" />  
][4]

That’s why Sysdig is continuing to add more Prometheus-related functionality to Sysdig Monitor - both the SaaS offering and the On-Prem software offering. Here are some of the things we have on our roadmap for this project:

*   Grafana support (in beta!) as this is the most common way people look at their Prometheus metrics

*   Expanded support of Prometheus metric types

*   Native use of a query language like PromQL

*   Easier migration between Prometheus and Sysdig Monitor

As you can see, we’re already working on a slew of functionality to continue to enhance our Prometheus integration. We’re really excited about this path of development for Sysdig Monitor, and will keep you updated as we progress, stay tuned!

 [1]: https://sysdigrp2rs.wpengine.com/product/monitor/
 [2]: /wp-content/uploads/2018/02/Prometheus-metrics-Sysdig-Monitor-UI.png
 [3]: https://sysdigrp2rs.wpengine.com/product/prometheus-support/
 [4]: /wp-content/uploads/2018/02/Prometheus-metrics-Grafana.png