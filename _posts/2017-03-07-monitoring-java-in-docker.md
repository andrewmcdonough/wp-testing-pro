---
ID: 3813
post_title: Monitoring Java in Docker at CDK
author: Apurva Dave
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/monitoring-java-in-docker/
published: true
post_date: 2017-03-07 22:04:15
---
#### Summary

The Digital Marketing business unit of CDK global shifted to a containerized approach for their next generation infrastructure. One of the challenges they ran into was how to monitor java applications in containers. Learn about some of their challenges and their use of Sysdig in this context.

#### About CDK

CDK Global provides integrated technology solutions to over 27,000 Auto, Truck, Motorcycle, Marine, Recreational Vehicle and Heavy Equipment dealers throughout North America, Europe, the Middle East, Africa, Asia Pacific and South America.

One of the many divisions of CDK Global provides Digital Marketing to their partners. Branden Makana, a senior engineer at CDK describes it, “ We provide the technology to our customers that enables an improved end-to-end customer experience. Think of a consistent, valuable experience from the website to the dealer lot and everything in between.”

#### Challenges: Big Data in Containers

Branden described his Business Unit at CDK in more detail. “CDK overall delivers the technology that vehicle dealers throughout the world need. My business unit is really into the personalized consume experience. Essentially you can think of us as the data analytics BU within the company. And, as a technology-forward company, we ideally wanted to have a common platform across BU’s for the next generation of our application infrastructure.

“This is challenging given how varied our offerings and applications are, but our data analytics BU was a great place to start. We had 2 environments, and two 2 data centers. We run beefy machines - 24 core, 256g ram with a single VM. Previously had a custom PaaS, but that wasn’t going to work for our next generation technology. So we started testing out new, containerized infrastructure using CoreOS. I’ll get to Docker in a bit, but first let me talk about our applications.”

CDK’s applications were heavily Java based. That’s not a surprise given the big-data focus of this business unit. Branden noted, “The applications we we build for data processing and ETL are web apps, using Java, typically with REST APIs, and Nginx front ends. We also have a number of PHP applications.”

Branden also described his team in more detail. “We have 10 development teams within our business unit. We all believe in using a DevOps model. That means that each team must be able to access their own monitoring data within our shared infrastructure. This played heavily into how we decided to instrument the system.”

#### Monitoring Containers

Our existing monitoring infrastructure used nagios for alerting on endpoints. Java Melody holds JMX metrics, and Graphite is our workhorse, holding 100,000+ stats. “While Graphite was open source, I know how much we had to put into our graphite infrastructure. It’s not cheap, and despite all that work, Graphite retention was really short - on the order of a couple weeks.

Branden recalled his first experience with Sysdig. “I ran across Sysdig at CoreOS Fest, and it seemed like a promising way to monitor containers. I knew we’d probably be better served with monitoring system that actually understood the additional complexity that came along with Dockerized infrastructure, so I figured we’d give Sysdig a shot.

“I was pleasantly surprised that Sysdig didn’t just give me a view of my Docker containers, but gave me a view into my docker containers. We could effectively see all the applications running inside the containers, and that makes Sysdig incredibly powerful."

Branden described the instrumentation process. “Sysdig really made metrics easy. From a development perspective we don’t have to do any work, just drop the agent in and go. Sysdig’s instrumentation collects hundreds of basic metrics from system calls - host resources, network utilization, container statistics, and application performance. As an added bonus, we don’t have to plug up Java Melody properties. It’s as self service as possible. There are no handoffs, or coupling, and that’s a huge productivity boost."

“Once we were up and running, I found that I spent a lot of time on the explore page… it’s an intuitive way to slice-and dice your application metrics when you’re trying to solve a problem. The other feature that has been really useful is grouping. We use docker labels to organize by data center, application, and service. Sysdig natively understands those pieces of metadata, and collects it all for us. That means, whenever we want we can switch views. I can quickly look at the performance of an entire data center, switch to an app-per-app view, and even isolate services."

“We’re also using dashboards more and more. There are very few monitoring applications that can get all your key metrics onto one page. Despite the massive amount of data, Sysdig performs well too. On top of all that Sysdig has unlimited data retention! I joke that Sysdig might have Hoffa’s body buried under our metrics somewhere.”

#### Monitoring Java in Containers

“One thing that surprised us is complete view that Sysdig gave us for our Java applications. For the first time we could see everything from the host to the network to the container, JVM, JMX, and application metrics in one place.”

Collecting JMX metrics within containers can be challenging, because you need to open specific JMX ports. Most of the time, that requires developers to tweak the container image to allow this. That means developers can't reuse off-the-shelf container images from the Docker Hub, and then they must propagate this information in whatever cluster management software is in use. This might mean simply exposing those ports in case of global polling or running sidecar containers, which then require additional resources and setup.

“Sysdig just sees the JMX metrics. It doesn’t matter where they are being written out from or to, Sysdig just sees them in system calls and collects them. That’s a huge boon to our development teams - we don’t have to go back and modify code dealing with our metrics, nor do we need to set up sidecar containers. This will work great in our next version of the PaaS as well, where developers might not have access to underlying infrastructure anyway.

“We’ve used Sysdig to troubleshoot a number of problems. For example, it was simple to find out why a container died - we saw performance drop and were able to correlate it with a Docker container out of memory event.

“That type of simple, intuitive troubleshooting saves us a lot of time and effort. It means we can stay focused on developing good code.

#### Advice

When asked for his advice on monitoring - java apps and otherwise - for containers, Branden noted a couple things.  [tweet_box design="default" float="none"]“First, you need to have a tool that can see inside containers, and can do it with little to no work on the part of your development teams.”[/tweet_box]

Second - make sure your monitoring tool is headed where you are. Branden described what he meant: “We’re looking forward to taking a more enterprise view with Sysdig,” Branden said. “As I mentioned, we’re building the next generation of our PaaS, and Sysdig will play a role in that. I especially like their new Teams feature, which will allow us to isolate data per development team or service. That’s particularly useful for when we have even more development teams working off a shared environment. It’s clear to me Sysdig is thinking carefully about the kinds of features I’ll need in the future and trying to stay in-line with me as we change.”