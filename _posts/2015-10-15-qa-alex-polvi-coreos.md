---
ID: 2162
post_title: 'Q&#038;A with Alex Polvi, CEO of CoreOS'
author: Chris Crane
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/qa-alex-polvi-coreos/
published: true
post_date: 2015-10-15 09:00:28
---
We recently sat down with <a href="https://twitter.com/polvi" target="_blank">Alex Polvi</a>, CEO of <a href="https://coreos.com/" target="_blank">CoreOS</a>, to pick his brain on our favorite topic: containers. Here at Sysdig, <a href="https://sysdigrp2rs.wpengine.com/coreos-sysdig-part-1-digging-into-coreos-environments/" target="_blank">we're huge fans of CoreOS</a> and everything they've contributed to the container community. So we're really excited to bring some of Alex's insights to you today. We managed to cover everything from the Open Container Initiative to CoreOS's commercial offering, Tectonic, to the future of the container industry. Read on and check it out! 

  
**Q: Earlier this summer, the Linux Foundation announced the [Open Container Initiative][1], which both Sysdig and CoreOS are members of. Where do you see the Open Container Initiative heading in both the short and long term? Is the availability of different container runtimes good for the industry and why?** 

**Alex:** The Open Container Initiative has been an important step forward for our industry to come together toward the creation of a shared standard for containers. We are hopeful that we can bring the best ideas together from across the industry in the design of it. While we still have a lot of work to do on open standards for containers, we are focused on bringing the expertise from the work on the App Container specification and include it as an important part of the OCI. 

A core tenet we are working to deliver in the OCI is that users should be able to package their application once and have it work with any container runtime (like Docker, rkt, Kurma or Jetpack). That means multiple implementations are necessary for the widespread success of the OCI. Work on rkt continues and we hope to make the rkt container runtime an implementation of OCI. We even recently introduced some increased security features in rkt 0.8, which included <a href="https://coreos.com/blog/rkt-0.8-with-new-vm-support/" target="_blank">initial support for user namespaces and enhanced container isolation using hardware virtualization</a>. 

  
**Q: During a recent talk at OpenStack Silicon Valley, you debunked several popular container myths. What are these myths, and why is it important to educate the market about these common misconceptions?** 

**Alex:** The promise of containers includes higher utilization of servers, high availability, failover and more. One of the most important aspects that has driven interest is that containers allow for greater consistency and interoperability from dev to cloud thanks to supporting the same open APIs. 

There are a few misconceptions about containers, one of them being that legacy apps do not work in containers. That is not true! At the end of the day, containers are a process running on a server just like anything else. There is no reason you can’t run a legacy application on a server. In fact, containers are delivering greater consistency between environments, so containers should enable companies to have more flexibility with where they run any traditional applications. Others myths we discussed were covered well by Sean Michael Kerner in <a href="http://www.datamation.com/open-source/container-myths-debunked-at-openstack-silicon-valley.html" target="_blank">Datamation</a>. 

We bet on containers at the inception of CoreOS in 2013, and now look at how quickly the container ecosystem has grown. We are committed to educating and training to help companies be successful with and learn from our expertise with containers. Adoption is increasing rapidly but continued education about the benefits will help more and more companies get containers running in production. 

  
**Q: What is the importance of monitoring and visibility in a containerized environment? What kinds of metrics should you be monitoring in production CoreOS deployments?** 

**Alex:** Monitoring containerized environments is hugely important, just like it is to monitor the health, state and utilization of any environment. We think CPU, memory, load average, network and disk utilization, and I/O are among the important metrics that should be monitored in production. 

  
**Q: How much progress are you seeing with regards to container adoption in production? What are the biggest barriers at this point?** 

**Alex:** Container adoption in production is about making sure companies have the knowledge in place to be successful with containers. Security and interoperability between environments are a big priority for production scenarios. We at CoreOS have built the pieces to run infrastructure in this way, which is what we are focused on packaging up in Tectonic and making it painless and easy to adopt containers in production. 

  
**Q: In April, CoreOS announced [Tectonic][2], the commercial Kubernetes platform. Tell us more about why you are building Tectonic, and the problems you are addressing.** 

**Alex:** Tectonic is our solution providing “Google’s infrastructure for everyone else.” Tectonic is about helping businesses getting the best compute infrastructure, just like Google and other hyperscale companies have used containers to run in a distributed, highly efficient and highly available way. Tectonic provides the combined power of the CoreOS portfolio and the Kubernetes project to any cloud or on-premise environment. 

Businesses want to securely run containers at scale in a distributed environment. We help them do this with the open source tools we’ve built. With Tectonic, we now have an option for companies that want a pre-assembled and enterprise-ready distribution of these tools, allowing them to quickly see the benefits of modern container infrastructure. Tectonic pre-packages all of the components required to build Google-style infrastructure and adds additional commercial features. These include the Tectonic Console for workflows and dashboards, an integrated registry to build and share Linux containers, and additional tools to automate deployment and customize rolling updates. 

Today <a href="https://tectonic.com/" target="_blank">Tectonic</a> is in Open Preview and during this period users are welcome to try it out at no cost and provide feedback to us leading into the general availability. 

  
**Q: What are the top container trends to keep an eye on heading into 2016 and beyond?** 

**Alex:** The world is excited about software containers, but there is still much more businesses want to do with containers in order to run them in production. One big piece of that is getting security built into container environments. And another is making sure that companies can easily achieve consistency between dev, cloud and data center. These are two areas we are working on closely at CoreOS and will be hot topics heading into 2016 and beyond.

 [1]: https://www.opencontainers.org/
 [2]: https://tectonic.com/