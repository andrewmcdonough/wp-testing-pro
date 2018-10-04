---
ID: 2066
post_title: >
  Sysdig Cloud Adds Support For Docker
  Labels
author: Marcus Sarmento
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-cloud-adds-support-for-docker-labels/
published: true
post_date: 2015-09-24 07:00:59
---
As the only container-native monitoring solution out there, we constantly strive to keep up to date with all of the ever-changing features and use cases for container technologies like Docker. Today I’m pleased to announce that we’ve expanded our native integration with Docker by adding comprehensive support for **Docker labels**. 

If you aren’t familiar with what Docker labels are, <a href="https://docs.docker.com/userguide/labels-custom-metadata/" target="_blank">Docker describes</a> them as metadata you can add to your images, containers, or daemons. There are a number of different use cases for Docker labels, but at a high level, they give Docker users the ability to tag containers with an arbitrary string of text. This helps you organize and manage your containers in the way that makes the most sense to you and your business. 

And now Sysdig Cloud automatically incorporates the full context of these Docker labels into your monitoring experience. No configuration needed! 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/09/docker-label-screenshot.png" alt="docker label screenshot" width="1500" height="839" class="aligncenter size-full wp-image-2072" />][1] 

Why does this matter? Monitoring and troubleshooting is all about context. Sysdig Cloud gives you hundreds of different metrics, and these are all automatically correlated to the metadata we’re pulling from your environment. So, on top of host names/ip addresses/etc, AWS tags/security groups/regions/etc, Docker images/container names/etc, and all the other meta data that we collect, you also have the added context of your Docker labels. 

Want to see rich metrics from inside containers -- metrics like SQL response times or Java heap usage -- charted in real time and broken out by Docker label? No problem! Want to alert on network traffic to a particular port of a particular type of container tagged with a certain combination of Docker labels? No problem :) 

Sysdig Cloud is the only monitoring solution on the market that can offer you anything like this visibility. This new support for Docker labels is just the latest improvement for our container-native solution, and I can’t wait to show you what’s coming next. If you haven’t done so already, I encourage you to <a href="https://sysdigrp2rs.wpengine.com/landing-page/?utm_source=web&utm_medium=blog&utm_campaign=dockerlabelblog092415" target="_blank">sign up for a free trial</a> of our container-native monitoring, alerting, and troubleshooting solution and try out our Docker label support yourself.

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/09/docker-label-screenshot.png