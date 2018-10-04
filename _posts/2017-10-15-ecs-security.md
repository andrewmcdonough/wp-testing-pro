---
ID: 4967
post_title: >
  How Sunrun Secures and Monitors its
  applications in Amazon ECS
author: Apurva Dave
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/ecs-security/
published: true
post_date: 2017-10-15 23:53:41
---
Implementing run-time Amazon ECS security as well as monitoring tooling for your containerized and microservice based infrastructure are two very large challenges. We caught up with John Hovell, Software Architect, and Andy Vansickle-Ward, Principal DevOps Engineer at Sunrun, to understand how they did it.



### About Sunrun



[Sunrun][1] is the largest dedicated residential solar company in the United States. In collaboration with our certified partner network, Sunrun helps homeowners get solar energy that is perfect for their home and lifestyle, simply and quickly.



Sunrun has a history of blending innovation with expertise. In 2007, Sunrun invented the “solar as a service” model and brings over 17 years of experience in designing and maintaining high quality solar systems—making clean solar energy affordable, mainstream and accessible for everyone in the U.S. Sunrun sells directly to consumers over the phone, web, and at retail stores, as well as through a network of certified partners. Sunrun has deployed more than $4.6 billion in solar systems. Sunrun has 100,000+ customers across the country and is growing quickly.



### What’s your Environment Like?



Our team is in charge of delivering our custom business applications. These are the apps that are visible to our customers and partners, so they are tied directly to revenue. They need to be reliable and highly performant. Our environment size today is in the range of dozens of VMs, and hundreds of containers. We’re just 3 full-time engineers to manage it all.



Our applications are built off what I’d consider pretty standard stuff for modern apps. We’ve taken a services-based approach to our app. We’ve got a sizable front end team building both web-based customer portals as well as mobile apps. We use Java for our backend applications and components.



All of this is on [AWS][2]. Amazon is our platform of choice - in order to be consistent we try to do everything with Amazon if we can. It’s good, cost effective, and reliable. So it shouldn’t surprise you that we also use AWS Lambda, and our containers are orchestrated by ECS.



### You’re using ECS. Tell us about that choice.



Absolutely. When we went head first into containers, it was a couple years ago. So we were really early by all accounts. At the time, we had a few choices out there. We looked closely at Kubernetes and ECS.



Even back then we realized Kubernetes was powerful, but it seemed much more complex than we needed. Given that we’re a small team of devops engineers, it didn’t seem to us that investing the time to learn, tune, and manage Kubernetes would have paid off. At the same time, ECS aligned with our philosophy of going AWS wherever possible. We found it easy to use, quick to get started, and with all the core features we needed. And, since it was all running on top of EC2 anyway, we understood exactly what our investment was going to be in running the entire setup.



### How do you get visibility into this environment?



This was one of the earlier challenges we ran into with our move to containerization. Legacy tooling would still give you a sense of how your hosts were doing, but that rapidly became low priority once you were running containers.



We realized that we needed something that understood containers, and could stitch together the data we needed across containers to give us a complete view of how a service was performing. So we did a search for tooling that could support containers. A few folks said they could, but in our trials Sysdig had the best service and container awareness.



A good example of that is the “grouping” capability in Sysdig Monitor. The tool automatically figured out that we were running in ECS, and then pulled the correct metadata such as tasks and services. From then on we could automatically aggregate data across all the containers that make up a service, no matter how frequently those containers would come and go. We could then build dashboards, implement alerts, and explore all this data in a service-centric way.



I should also mention that the underlying instrumentation to all this was nonintrusive. We simply had to deploy an additional container to each host - that’s Sysdig’s agent - and then Sysdig figured out the rest, including what applications were running inside the container. That was really useful because, when ECS needs to move or reprovision containers, Sysdig automatically knows what application metrics to capture without one of us doing manual configuration.



### How do you secure this environment?



We have a great corporate security team, so they gave us strong walls of protection to begin with. But - much like monitoring - we saw that with the move to containers we have an opportunity to potentially add a new layer of run-time security that is even more intelligent that what we had previously. We thought we could implement a system that would alert us when any anomalous behaviors were found in the system and then take action as a result.



In the last year or so container security approaches have evolved, so we took a look at just about everything out there. In doing so, we noticed a few things. While there were container security approaches, none of them really could up-level to the “service,” and that’s of course the thing we care about now - containers are somewhat irrelevant. And nothing really focused on unifying all the data we’d want to see about our environment, whether its performance data or events, either of which could indicate a potential problem with our system.



That’s when we learned that Sysdig was extending their platform to security. We knew that their form of instrumentation could potentially give us great visibility into the security of our services in addition to the performance data we were already getting, so we decided to give it a shot.



### Can you tell us about the process of going from Sysdig Monitor, to using the Container Intelligence Platform with Both Sysdig Secure and Monitor?



Let’s talk about instrumentation first. We were really happy to see that we didn’t have to add any additional instrumentation beyond the Sysdig agent we already had. All we had to do was flip a flag in the agent configuration file, and then Sysdig gave us access to a new, Security-focused UI. That was a huge win, not only for simplicity, but also resource usage. A single agent per host could be saving us anywhere from 5-10% of our resource cost per host. That’s a pretty big deal.



Once we got into Secure, we felt right at home. The shared UI design between Monitor and Secure meant that we immediately understood the application, and things were right where we thought they would be. Perhaps more importantly, Sysdig carried over its ability to group security policies and events by physical infrastructure or by logical ECS metadata.



This is exactly what we wanted - a services-oriented way to manage security policies. Now if we have an issue, we can easily understand the services impacted and use the UI to scope and filter the data on that set of containers, no matter where they are distributed.



The policies themselves are easy to modify. As with almost any security product, we spent a bit of time up front modifying the policies to match what we care about. But there was a low learning curve for Sysdig. It helped that the product had a lot of strong policies out of the box, such as when a database opens an outbound connection, or a user spawns a shell within a container.



All in all, we found that Sysdig is the only one who has unified performance monitoring and security, and done it in a low-resource and cost effective way.



### What’s next?



One feature we haven’t used quite as much yet are Sysdig Captures. They take a snapshot of an environment’s system calls when an alert is triggered (in Monitor) or a policy is violated (in Secure). It seems like a really powerful way to analyze anomalies get to the why of the problem. Sysdig has just released a new interface for inspecting captures, so we want to empower the team to take advantage of this capability. We’re going to be tuning our workflow to take advantage of this additional tool at our disposal.

 [1]: https://www.sunrun.com/
 [2]: https://aws.amazon.com/