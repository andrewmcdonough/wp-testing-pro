---
ID: 2542
post_title: How to visualize DC/OS
author: Apurva Dave
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/visualize-open-dcos/
published: true
post_date: 2016-04-17 22:09:50
---
We’re excited to see that DC/OS platform has [gone open source][1]. What’s more we’re fortunate enough to have been invited along for the ride! This post will give you a short overview of what’s happening, why it matters, and how Sysdig is involved.

### What is DC/OS?

DC/OS (Data Center Operating System) was originally created by Mesosphere. It takes two key open source projects, Mesos and Marathon, and enables you to abstract away distributed resources into one “pool” of memory, compute, and disk. DC/OS bundles additional services such as mesos-dns, tooling like a CLI, a GUI, a repository for the packages that you want to run, and frameworks like Marathon (a.k.a. distributed init), Chronos (a.k.a. distributed cron), and more. The end result is a simple, packaged way to run complex distributed applications with or without containers.

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/04/dcos1_7_dashboard.png"></a>[![DCOS Dashboard][2]][2] 

*All the power of DC/OS, now open source! * 
### Why does it matter?

As most of you know we’re currently amidst a major platform shift - the move from virtual machines to containers will allow organizations to deliver more functionality, faster. Projects like DC/OS matter because they simplify the process of getting an environment running that can effectively manage and enable containers in the first place. In short, containers are great, but only if you have something like DC/OS to take your containers into production.

### But you’ve still got to monitor it all!

It was just a couple weeks back that Sysdig [announced its formal partnership with Mesosphere][3], and more importantly, our [technical integration with Mesos, Marathon, and DC/OS][4].

And because the open sourced DC/OS is full-featured DC/OS, our integration works exactly the same across our open source and commercial products. The link above has an example of deep troubleshooting… instead of recreating that I’ll provide a “getting started” example.

### Visualizing DC/OS at Work

Deploying applications with DC/OS is a totally new paradigm, so it can be challenging to wrap your head around what exactly the platform is doing. How is it allocating resources? Where is your application actually running?

Let’s suppose you’re deploying a few applications on top of DC/OS and you’d like to understand: 
*   What are the applications?
*   What resources are they using?
*   How are they performing?

Enter Sysdig Cloud.It provides real-time topology mapping that allows you to automatically visualize the physical resources at work. You simply use our one-line command to deploy the Sysdig agent. No additional Sysdig Cloud agent configuration is required when the Mesos master is installed with default settings on the nodes where the Mesos API server runs ("Master" nodes). The Sysdig Cloud agent will look for the process named “mesos-master”. If the process is found at any time, our agent will automatically connect to the local Mesos and Marathon (if available) API servers, to collect cluster configuration and current state metadata in addition to host metrics.

OK, so point-and-click, you’ve pushed out your applications on DC/OS. 

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/04/dcos1_7_services.png"></a>[![DCOS Dashboard][5]][5] 

*Services view within DC/OS*   
  
Assuming you’ve instrumented the hosts running DC/OS, you can get a view like this which tells you how the machines are doing from an infrastructure perspective:

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/04/Mesos-Topology-1.png"></a>[![Sysdig Mesos Dashboard][6]][6] 

Nothing too unusual (or insightful) here… this is just straight infrastructure monitoring. But let’s start to look at relationships of these hosts in a topology:

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/04/Mesos-Topology-2.png"></a>[![Sysdig Mesos Dashboard][7]][7] 

OK, so this is slightly more interesting. We can see communication between the hosts, request times, and resource usage of the machines themselves all in one place. But how does this relate to your app? And what is DC/OS doing as a part of this? Let’s drill down into these hosts…

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/04/Mesos-Topology-3.png"></a>[![Sysdig Mesos Dashboard][8]][8] 

And we see small boxes that each represent a container running its own software… it’s spaghetti! And that’s a good thing, it means DC/OS is doing its job. DC/OS is distributing containers across machines in a way that makes the optimal use of underlying resources. 

As we mentioned, Sysdig also integrates with the master to collect metadata about your system. We use that data to take physical data, like what you’re seeing above, and create logical picture of what your apps are doing. When you apply the metadata, you get this:

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/04/Mesos-Topology-4.png"></a>[![Sysdig Mesos Dashboard][9]][9] 

OK, now we’re cooking. We can see a few applications, as well as the relationships between their components, as well as the resources they are using. While the combination of physical and logical topology is especially useful for troubleshooting, it’s also a huge help in understanding the basics of how DC/OS works in the first place.

### Just the beginning…

Deploying a containerized application is just a step in your overall process from conceptualizing an application all the way through running it in production. DC/OS makes it simple to get those apps running, and Sysdig is very happy to partner with the DC/OS team to make monitoring just as simple. While in this post we focused on visualizing how an application is deployed on DC/OS, our next post will focus on what your application is doing and how you can troubleshoot it more effectively. Stay tuned!

 [1]: http://dcos.io/
 [2]: /wp-content/uploads/2016/04/dcos1_7_dashboard.png
 [3]: https://sysdigrp2rs.wpengine.com/press-releases/sysdig-mesos-partnership
 [4]: https://sysdigrp2rs.wpengine.com/blog/monitoring-mesos/
 [5]: /wp-content/uploads/2016/04/dcos1_7_services.png
 [6]: /wp-content/uploads/2016/04/Mesos-Topology-1.png
 [7]: /wp-content/uploads/2016/04/Mesos-Topology-2.png
 [8]: /wp-content/uploads/2016/04/Mesos-Topology-3.png
 [9]: /wp-content/uploads/2016/04/Mesos-Topology-4.png