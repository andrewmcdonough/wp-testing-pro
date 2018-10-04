---
ID: 2123
post_title: >
  Announcing Microservice-Level Monitoring
  for Amazon’s ECS
author: Loris Degioanni
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/announcing-microservice-level-monitoring-for-amazons-ecs/
published: true
post_date: 2015-10-06 06:00:43
---
Containers are taking the world by storm. Here at Sysdig, we believe containers are the next great infrastructure platform. And this vision is being realized through the ecosystem of technologies that has been developing around core tools like Docker and CoreOS. Amazon’s EC2 Container Service (ECS) is an interesting and exciting example: ECS offers container orchestration as a service, on top of the traditional AWS infrastructure. ECS has been designed to support microservice-based applications, and makes it really easy to run and scale a Docker-based infrastructure. 

There’s a catch though: with the extra layers of abstraction and orchestration comes increased complexity and decreased visibility. Containers are by their nature opaque and impractical to instrument. When you add in orchestration, the containers running each microservice will be scattered around and intermingled throughout your infrastructure. Not exactly the easiest situation to try to troubleshoot in! The reality is, until now, the ability to monitor ECS-based environments has been very limited, making running a production application a hard proposition to get comfortable with. 

Today I’m pleased to announce that Sysdig Cloud can now help you deploy your ECS-based applications with confidence. We’re updating Sysdig Cloud to offer the first and only comprehensive monitoring solution for Amazon ECS. 

We are bringing to you, for the first time, full ECS visibility, including the ability to monitor and alert at any level of the application stack, from services and tasks (see below if you’re not familiar), down to hosts, containers and processes. All of this with zero configuration and no need to instrument any of your containers with agents. 

## ECS 101

If you are not familiar with ECS, <a href="http://www.allthingsdistributed.com/2015/07/under-the-hood-of-the-amazon-ec2-container-service.html" target="_blank">this blog post from Amazon’s CTO Werner Vogels</a> has a good architectural description. ECS offers a way to coordinate containers by grouping them into **tasks** and **services**. 

A task is essentially a single unit of a microservice. Each task is often, but not always, a single container image, and could be, for example, a web server or a data store. As shown in the picture below, tasks are defined through the AWS console, starting from Docker images. 

<a href="/wp-content/uploads/2015/10/AWS.png" data-rel="lightbox-0" title=""> <img src="/wp-content/uploads/2015/10/AWS.png" class="aligncenter size-full" /> </a> 

Once you have defined your tasks, you can include them in a service, which makes it possible to specify how many copies of a task should be running and how they should be allocated. Once the service is started, ECS takes care of the scheduling and load balancing for you. 

Containers belonging to the same service or task can be run across arbitrary machines, intermingled with other services and tasks, and linked up through multiple layers of abstraction. This is great for resource utilization, but it makes capacity planning, performance management, troubleshooting, and visibility in general really challenging. 

## Microservice Monitoring With Sysdig Cloud

So what exactly can Sysdig Cloud do for you in your ECS environment? For that matter, what should you be looking for in any monitoring tool that claims to support ECS?

At a high level, Sysdig Cloud offers two things: 

1.  By extracting metadata from Docker and ECS, we’re now able to infer the logical structure of your microservice application.
2.  This is information is combined with our patent-pending ContainerVision technology, which makes it possible to inspect applications running inside containers without requiring any instrumentation of the container or application.



The result is rich visibility from both an infrastructure and application centric point of view.

Let’s dig into what this actually means for you!

Sysdig Cloud now lets you group and explore your containers according to the **physical hierarchy** (for example, container > EC2 instance > availability zone > region) or according to the **logical microservice hierarchy** (container > task > service > application > cluster), as shown in this screenshot. 

<a href="/wp-content/uploads/2015/10/Semantic-Explore-ECS.png" data-rel="lightbox-1" title=""> <img src="/wp-content/uploads/2015/10/Semantic-Explore-ECS.png" class="aligncenter size-full" /> </a> 

The physical view is great to understand how your resources are utilized, and if a container is slowed down by noisy neighbours. However, the logical one is the perfect starting point to inspect the **performance of your microservice**. This is, of course, how most operators think about their applications - and amazingly, no other monitoring tool can yet provide this application-centric point of view in ECS. That’s the power of ContainerVision. 

Here, for example, I’m observing the overall performance of a Wordpress task. 

<a href="/wp-content/uploads/2015/10/URLs.png" data-rel="lightbox-2" title=""> <img src="/wp-content/uploads/2015/10/URLs.png" class="aligncenter size-full" /> </a> 

Note how the containers implementing this task are running on multiple machines, but I can still see an aggregate of the CPU, response time and slow URLs for this service. And remember: this **doesn’t involve any kind of instrumentation of either Apache or the containers running it**! Of course, I can easily create an alert for any of the metrics shown on the screen. And of course I can easily drill into a single Apache container and get rich visibility inside it, down to the process level. 

## Visualizing Your ECS Microservice

Oh, and one more thing. :-) 

We’ve updated our award-winning topology representation, so you can also visualize the logical topology of your microservice application. Check out the two pictures below: they show the same service, but the first one depicts the physical hierarchy, while the second one groups containers into tasks and services, ignoring their physical location. 

<a href="/wp-content/uploads/2015/10/Recording-132.gif" data-rel="lightbox-3" title=""> <img src="/wp-content/uploads/2015/10/Recording-132.gif" class="aligncenter size-full" /> </a> 

<a href="/wp-content/uploads/2015/10/Recording-133.gif" data-rel="lightbox-4" title=""> <img src="/wp-content/uploads/2015/10/Recording-133.gif" class="aligncenter size-full" /> </a> 

Hopefuly it’s evident how much more natural and intuitive the second representation is. You can easily follow the structure of the application and visually identify any hot spots. **The interactions between microservices become obvious, even though these microservices and their components are scattered arbitrarily across the infrastructure on ECS!** 

## Conclusion

Monitoring microservices, especially when based on sophisticated orchestration tools like ECS, is not just a matter of collecting more metrics. It’s a matter of extracting the right information, without friction, and presenting it in a way that reflects the services’ architecture and interactions. Sysdig Cloud was built from the ground up to provide exactly this type of visibility for the new world of containers and microservices. We’re constantly adding new, cutting-edge support for the latest and greatest technologies emerging in this ecosystem. And we’re excited to now be supporting the many organizations making the jump to Amazon ECS. This is game changing technology that deserves to be used in production, and we want you to innovate and adopt with confidence.