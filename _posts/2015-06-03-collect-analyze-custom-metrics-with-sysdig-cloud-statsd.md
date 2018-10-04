---
ID: 1596
post_title: 'Collect &#038; Analyze Custom Metrics with Sysdig Cloud &#038; StatsD'
author: Marcus Sarmento
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/collect-analyze-custom-metrics-with-sysdig-cloud-statsd/
published: true
post_date: 2015-06-03 06:00:06
---
I’m pleased to announce that another one of our most requested features has been released - support for <a href="https://github.com/etsy/statsd" target="_blank">StatsD</a>. Now Sysdig Cloud makes it really easy to collect custom application metrics and gain deep insights into complex environments by leveraging our painless setup, zero config container support, and automatic correlation of metrics from across your infrastructure. 

## Painless Setup 

We wanted to make it REALLY easy to use our StatsD support with minimal configuration effort. If you already have StatsD setup in your environment, the Sysdig Cloud agent will automatically see and collect StatsD metrics being sent to your StatsD server without needing to install any plugins or configure the agent. We also pull in the associated groupings and hierarchies automatically so you can segment the StatsD data by group or by host for example. So if you’re interested in trying Sysdig Cloud out with your existing StatsD deployment, you don’t have to do anything besides installing our agent on the host OS. It doesn’t get much easier than that, right? 

Of course, if you’re not already running StatsD, it’s very easy to get going with Sysdig Cloud. With a few simple tweaks, you can send StatsD metrics and additional tags to the Sysdig Cloud agent directly (which would then act as the StatsD server). 

## Zero Config Container Support 

As more and more organizations migrate to containerized environments, we thought it was really important to have zero configuration StatsD support for containers right from the start. With this release, we allow you to consume StatsD metrics from **inside containers** with **no instrumentation**, **no need for a StatsD server in the container**, and **no need for network tuning**. No other container monitoring technology can deliver on these promises. We even let you create additional tags for container properties and allow you to segment by those new container-related tags automatically. So, if you already use containers (or will be using them soon) together with StatsD, Sysdig Cloud has you covered. To learn more about the technical details that make this awesomeness possible, <a href="https://sysdigrp2rs.wpengine.com/see-statsd-custom-metrics-inside-containers-automagically-with-sysdig-cloud" target="_blank">check out our technical StatsD blog post</a> we published. 

## Auto-Correlation To Deep Infrastructure Metrics 

Not only do we make it really easy to use Sysdig Cloud with your existing StatsD implementation, but we automatically correlate all of your custom application metrics from StatsD with rich, one-second granularity metrics from across your environment. Example metrics you’ll get by default out of the box include: 
*   **System** (CPU, Memory, Disk Usage, etc.)
*   **Application** (JMX, HTTP, Status Codes, etc.)
*   **Infrastructure** (SQL, MongoDB, Redis, AWS Services, etc.)
*   **Network** (Traffic, Connections, etc.)
*   **Containers** (Docker, CoreOS, LXC, etc.)

Imagine being able to correlate things like revenue with application response times and CPU usage spikes with active users. Starting today, you can do exactly that with Sysdig Cloud. 

## Conclusion 

To summarize, we’ve made it incredibly easy to start collecting and analyzing your StatsD custom application metrics, automatically, with no configuration (even metrics inside containers!). Seeing your custom metrics alongside deep infrastructure metrics gives you the full context you need to make data driven decisions about your environment in real-time. <a href="https://sysdigrp2rs.wpengine.com/landing-pitch2/?utm_source=web&utm_medium=blog&utm_campaign=statsdannouncementblog" target="_blank">Sign up for Sysdig Cloud today</a> to try out our StatsD support out for yourself and let us know what you think!