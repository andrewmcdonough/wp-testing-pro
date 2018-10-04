---
ID: 3076
post_title: >
  Monitoring Docker on GKE and AWS at
  Everalbum
author: Apurva Dave
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/monitoring-docker-gke-aws-everalbum/
published: true
post_date: 2016-09-27 06:44:17
---
Everalbum transitioned from a more static, VM-based architecture to a Docker-based model across dozens of hosts and multiple cloud providers. In addition to monitoring Docker containers, Everalbum leverages Kubernetes and cloud provider-run services. They needed a solution that covers server monitoring, container monitoring and even custom statsd metrics from their application code. Read on to learn about the challenges Everalbum faced during this transition and how they used Sysdig to assist in the change.

### About Everalbum

Based in San Francisco, California, Everalbum is a data-intensive, mobile-first service designed to protect your most valuable memories. Everalbum protects your life's photos so you never have to worry about losing them. Across devices and photo sources, Everalbum automatically backs up your photos and videos so you can access them at any time. You can then free up space on your device by removing photos from your camera roll.

### Transitioning to containers

Like many applications today, looks can be deceiving. What appears to be a simple app on a user’s phone can (and should!) hide the complexity of the backend. That’s certainly true for Everalbum: the need to consistently and reliably store their users’ photos from any number of devices requires them to have a robust, distributed backend. It had to have high uptime, scalability, and the ability to roll out new features quickly.

In order to achieve this, Everalbum now runs services both on Amazon Web Services (AWS) and Google Container Engine (GKE), as well as Heroku. They have dozens of hosts that they monitor, in addition to services operated by the cloud providers themselves.

“About a year ago we started to make the transition to containers,” noted Jon Mumm, co-founder at Everalbum. “It just so happened we ran across Sysdig at the same time.” Jon recalled that they were looking at Docker containers for their potential to accelerate development and deployment of new code.

“We realized that docker monitoring would be different on a number of levels, and we wanted to get ahead of that problem. We knew that services like AWS Cloudwatch and Google Stackdriver were still fine for low level stuff, but anything that we were running inside a container would be an entirely different story.”

Everalbum uses Kubernetes to orchestrate containers across their environment, adding another important element to their monitoring and operations strategy.

### Requirements

Given Everalbum has a reasonably complex backend, they needed monitoring capabilities that could adapt. In particular they needed:

*   **Docker monitoring support:** They required a system that could do more than just aggregate CPU/Memory/Network stats from the Docker API and could give insight into applications running inside their containers.
*   **Support for non-containerized applications:** Certain stateful services don’t run inside containers, and Everalbum needed to monitor those as well.
*   **Kubernetes integration:** Kubernetes understands both the physical resources used by containers but also the logical services that the containers represent. Everalbum required a container-native monitoring tool must be able to tap into that knowledge.
*   **Simpler config for dynamic services:** Everalbum knows less work is better, especially in a Kubernetes world where containers can come and go at any time of the day.
*   **Custom metrics:** There was a heavy bonus for the monitoring system to capture statsd metrics as well so that they have one less monitoring tool they need to worry about.

### Sysdig for Container-native monitoring

Jon recalled his earliest experiences with Sysdig. “At the time we had 6 or 7 HTTP services, and it was a pain to constantly be tuning our monitoring system when there were changes. I was really impressed with Sysdig’s instrumentation: it automatically figured out where my services were running, what the services they actually were, and what the appropriate metrics were to collect. Sysdig could do this even if my services were in containers, without agents inside the containers.

“I also really liked the statsd capability. You can just send stuff to localhost and never worry about it - Sysdig just finds it. It’s an easy implementation that ensures the development team will take advantage of it,” said Jon.

Sysdig’s Kubernetes integration allows Everalbum to easily monitor at both the physical and the logical layers with their application. “The Kubernetes integration feels very natural. I can switch to a real-time logical view of my application using Kubernetes’ metadata for grouping containers. It also allows us to divide up monitoring for our dev and prod namespaces. They’re both important to monitor, but we treat them differently. Sysdig makes it easy to do that.”

### Monitoring HTTP endpoints

Jon talked a bit about being able to monitor applications and services, and not just Docker infrastructure metrics.

“For example, we keep a close eye on particular HTTP endpoints as early-warning indicators of application problems. Sysdig provides out of the box HTTP metrics, which make it really easy to get going. If you combine that with a service-level view using Kubernetes metadata, then you have a good sense of how your service is performing. That’s a pretty powerful combination of capabilities.”

Jon used to depend on CloudWatch for alerting on HTTP errors. “We would use CloudWatch whenever we saw a 500 on our ELBs. But CloudWatch would never resolve when the 500 count goes to zero. It sounds basic, but simple things like this make our lives easier. Sysdig figured this out of course.”

Jon noted where he wanted to take this functionality. “In addition to Sysdig reporting on the top HTTP endpoints and the slowest HTTP endpoints, they also allow measurement and reporting of any path in the system. We’re going to implement that soon to give us more coverage.”

### Monitoring Services that aren’t containerized

“We run a bunch of our applications on top of GPUs,” Jon pointed out. “For most of this stuff, we typically depend on our Statsd metrics, in addition to low level resource utilization metrics. Sysdig’s statsd capability allows us to aggregate all our Statsd metrics in one place for easy viewing and analysis.”

In addition, Everalbum runs Postgres. “We don’t run this stateful service in Docker. But that’s fine in terms of Sysdig.” Sysdig automatically detects Postgres whether or not it’s running in a container, and then collects the appropriate app-level metrics by enabling the correct application check.

### What the future holds

Jon noted that he’s seen continually increasing adoption of Sysdig across the company. “At first it was just me, but then when we started migrating alerts to Sysdig we saw more pickup. As we exposed more and more statsd metrics there’s also a pickup. Basically, as we leverage more and more functionality within Sysdig we’re seeing more adoption. We want to keep that moving in the same direction.”