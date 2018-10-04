---
ID: 1856
post_title: Monitoring as a Microservice
author: Loris Degioanni
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/monitoring-as-a-microservice/
published: true
post_date: 2015-07-14 06:50:03
---
Launch day. For an entrepreneur, this is always an exciting day. But this time I’m particularly excited. I’m excited because the world of software is changing, and now Sysdig gets to play a small role in the revolution. 

A paradigm shift is occurring in the way we create, deploy, scale and maintain our applications. Gone is the monolith. Modern applications are decomposed, distributed and API-driven. And this is not just the exclusive domain of internet giants any more. Microservices and continuous delivery are becoming the standard, not the exception, even in smaller organizations. And now we are witnessing the meteoric rise of the next generation of infrastructure that is enabling this revolution: **containers**. 

 

Thanks to projects like Docker, CoreOS, kubernetes and Mesos, containers have quickly evolved from glorified virtual machines into the platform on top of which modern software is being built. Containers are taking the world by storm, and causing a shift in the industry whose magnitude can be compared to the sea changes brought to us by personal computers, virtualization and cloud computing. Only thing: this time it’s happening much faster. And by the way, you’re not immune to it. Everyone from startups to Fortune 500 enterprises is investigating and adopting containers. Big change is afoot. 

And as with every major platform change, the container revolution requires new ways of doing things. Remember how infrastructure components like storage, networking, and security were totally rethought when the world switched from physical to virtual machines 15 years ago? If you want to unlock the true potential of a new platform, then you must have an ecosystem of native technologies to support it. Today, infrastructure tools across the board are being rebuilt from the ground up to be container-native. The speed, complexity, and dynamism of containerized environments demand it. Across our industry brilliant minds are hard at work on these new breeds of technology, but one area is still sorely lacking: visibility. Today, I am <a href="http://www.businesswire.com/news/home/20150714005452/en/Sysdig-Announces-10.7-Million-Series-Funding-Launches#.VaUUVJNVikq" target="_blank">proud to announce the launch of Sysdig</a>, the container visibility company! 

 

My colleagues and I at Sysdig have been building visibility and performance management tools for most of our lives, and we couldn’t pass up the opportunity to be part of this revolution. So over a year ago, we went to the drawing board, and we tried to imagine what **container-native visibility** would look like. And then we started building. Over the past year we’ve iterated and improved based on the invaluable feedback of our thousands of beta users (we love you!). And today, with over 30 paying customers (we love you too!), we unveil Sysdig Cloud, the synthesis of our vision and the result of our hard work. 

Sysdig Cloud is the first and only container-native monitoring, alerting, and troubleshooting solution, designed from the ground up to provide unprecedented visibility into modern, containerized infrastructures. Powered by Sysdig’s patent-pending ContainerVision™ technology, Sysdig Cloud can monitor your entire architecture from its own stand-alone container, with zero configuration, and yet still offer deep visibility into every other service running in your environment. 

Sysdig Cloud includes everything you would expect from a modern visibility solution: robust dashboards, intelligent alerting, comprehensive integrations. But it also sports some very unique, container-native, microservice-oriented features: 

*   Real time data with ten second granularity
*   Back in time DVR system replay
*   Dynamic topology mapping
*   Network traffic data from inside containers
*   Custom <span style="font-family:courier">statsd</span> metrics from inside containers
*   Application polling and health checks inside containers
*   Deep integration with container technologies like Docker and CoreOS



 

And just to reiterate, all of this requires no agents, no code instrumentation, no plugins -- nothing to pollute your container images in any way. Yes, it’s possible :) And this is what I find most exciting about our technology. Sysdig Cloud leverages containers to **reinvent the basics of monitoring**. Gone are the times when monitoring a new service required instrumenting code and installing agents. Gone are the times when forgetting any step meant a dark spot in your monitoring system. With Sysdig Cloud, you **just add an extra container**, and you suddenly see everything. Apps, services, networks, processes, infrastructure components. Even if they are there for only a few seconds. Now, and forever into the future. 

We call it **monitoring as a microservice** - it is the core of our vision, and it is designed to give you peace of mind. 

I’m also proud to announce today that some of our favorite people - people with a lot of experience at this kind of thing - have decided to join us in our mission. Accel Partners and Bain Capital Ventures <a href="http://www.businesswire.com/news/home/20150714005452/en/Sysdig-Announces-10.7-Million-Series-Funding-Launches#.VaUUVJNVikq" target="_blank">have invested a total of $13M</a> in our team, and we’re thrilled to have them on board. We’ll be devoting these funds to scaling our team and making our vision of container-native visibility into a reality. 

 

Are you interested in trying Sysdig Cloud? Nothing could be easier: sign up for a <a href="https://sysdigrp2rs.wpengine.com/landing-page" target="_blank">14 day free trial</a>, deploy our container, and experience comprehensive visibility. I promise it will take less than two minutes.