---
ID: 6782
post_title: >
  5 changes containers bring to PCI
  compliance.
author: Knox Anderson
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/container-pci-compliance/
published: true
post_date: 2018-03-28 22:26:49
---
Containers have been adopted faster than any previous enterprise technology, and for good reasons. They’re portable, provide robust security through isolation, and allow application teams to develop better services faster. However, the quick rise in adoption is a pace that’s hard to match on the compliance side. In this blog we’ll cover the main challenges and opportunities you need to be aware of to successfully manage Kubernetes and container PCI Compliance 

Before we get into that, we have some news around Sysdig’s commitment to helping organizations adapt to changing application lifecycles and achieve container PCI Compliance.

Sysdig has joined the PCI Security Standards Council (<a href="https://www.pcisecuritystandards.org/" target="_blank">PCI SSC</a>) as a new Participating Organization. We will work with the PCI SSC to help secure payment data worldwide through the ongoing development and adoption of the PCI Security Standards. We decided to join the organization so that we could bring our specific knowledge around operating containers in production to assist the PCI community as enterprises transition to next generation container platforms. 

Now let’s get into the fun stuff: the challenges and opportunities containers will bring to your organization's compliance initiatives. In this post we will break down:

*   Why will containers impact PCI compliance initiatives?

*   What complicates Kubernetes PCI Compliance?

*   How can I address individual PCI Requirements as my organization moves to containers?

## Container PCI Compliance {#pcicomplianceincontainers}

### Challenge 1. The number of entities you’re monitoring just increased by 10x {#challenge1thenumberofentitiesyouremonitoringjustincreasedby10x}

Last year we published our <a href="https://sysdigrp2rs.wpengine.com/blog/sysdig-docker-usage-report-2017/" target="_blank">Docker Usage Survey</a>, where we took an anonymous snapshot of a subset of containers under our management, and analyzed usage patterns of a 45,000 containers. One of the interesting takeaways was on average each host had 10 containers running. 

What does this mean for container PCI Compliance? If you used to have just 3 nodes with 10s of connections, they have now turned into 30 containers with hundreds of connections. Managing controls of network diagrams like PCI DSS 1.1.2 just got a bit more challenging. Let’s take a look a little more closely at PCI DSS 1.1.2:

**1\.1.2**** Requirement -** Current network diagram that identifies all connections between the cardholder data environment and other networks, including any wireless networks.

[Sysdig][1] will dynamically generate topology maps of all hosts, containers, and processes on your infrastructure and map any network connection they make inside and outside your network. These topology maps can also be customized to show the logical services and how they’re connected as well. 

![Network Map Container PCI Compliance][2]

*This Sysdig screenshot shows a host based view of a kubernetes cluster and all the local and remote connections.*

![Network Map Docker PCI Compliance][3]

*This second Sysdig screenshot shows how much more detail is now shown once a host is expanded and the containers running inside the host are exposed.*

Managing and meeting compliance controls like 1.1.2 is just one of the many examples we cover in our [Guide to PCI Compliance in Containers][4].

### Challenge 2. Release velocity increases and container lifecycles decrease {#challenge2releasevelocityincreasesandcontainerlifecyclesdecrease}

While automation has been prevalent for many years, the adoption of containers and orchestrators accelerated the rate at which computing entities come and go. It’s not uncommon to see small kubernetes deployments cycle through thousands of containers over a two week period, with often no more than 5 or 6 unique containers running at any given point in time. Having systems to auto discover containers being started, and then storing all the data about what, where, and when is crucial from an auditing perspective.

**2\.4 Requirement** - Maintain an inventory of system components that are in scope for PCI DSS.

Sysdig comes with an explore view that will give you a view of all hosts, containers, or any process grouped by metadata running on their system. They can use this table to slice and dice all system components however they choose.

![Auditing containers PCI compliance][5]

*By using the time controls at the bottom of the table users can always see what containers were running on any area of your physical or logical infrastructure at any point in time. These can also be filtered with custom tags to isolate containers within and outside the scope of PCI.*

### Opportunity 3. Containers are easier to baseline than VMs. Hooray! {#opportunity3containersareeasiertobaselinethanvmshooray}

One of the core architectural designs of a container is built in process isolation. A container should have one process running inside it, with expected behavior and performance. This clear behavioral profile makes it easier to build models and design security to look for anomalous activity inside the container, or how the container is interacting with the rest of your services.

### Challenge 4. The UserID 0 Challenge {#challenge4theuserid0challenge}

There are many compliance controls for capturing user behavior around privilege escalation, data access, or in general any command executed in an PCI scoped environment. The challenge with containers is every command is executed as root, making it much harder audit user behavior. Security auditing tools must now know how a user gets access to a container and correlate that with what is executed inside the container. 

**10\.2.2 Requirement Description -** All actions taken by any individual with root or administrative privileges

Sysdig by default will capture every action taken by a user on your hosts and inside your containers. These actions can also be viewed based on any piece of host, container, or orchestration metadata to view how commands can trigger lateral movement across your infrastructure. 

![Kubernetes PCI compliance][6]

*With Sysdig Secure auditors can browse commands executed based on any logical tag associated with a host, container, orchestrator, or cloud service.*

### Challenge 5. Treat orchestrators differently than containers {#challenge5treatorchestratorsdifferentlythancontainers}

Managing Kubernetes PCI Compliance is different than enforcing controls to individual container images. The rise of opensource software has led to the same images being used across multiple services. The policies that your team configures should take the entire service into account by using orchestrator metadata to enforce compliance on a service by service basis.

![pci compliance in containers][7]

*Sysdig Secure makes it easy to build policies based on the scope that is relevant to your PCI controls. Here we’re looking at specific policies that apply to a logical kubernetes namespace.*

Hopefully these quick points help provide some additional context around what to expect when adopting and operating containers in production environments. To learn more about <a href="https://sysdigrp2rs.wpengine.com/product/secure/" target="_blank">Sysdig Secure</a> and the <a href="https://sysdigrp2rs.wpengine.com/product/how-it-works/" target="_blank">Sysdig Container Intelligence platform</a> can help with your PCI Compliance initiatives check out some of these resources below.

*   Download our[ Guide to PCI Compliance in Containers][4]

*   Attend our webinar: [5 reasons why containers change PCI compliance][8]

*   Sign up for [Sysdig Secure Demo][9]

 [1]: http://sysdigrp2rs.wpengine.com/product/secure
 [2]: /wp-content/uploads/2018/03/PCI-compliance-in-containers-1.png
 [3]: /wp-content/uploads/2018/03/PCI-compliance-in-containers-2.png
 [4]: https://go.sysdigrp2rs.wpengine.com/PCI-Compliance
 [5]: /wp-content/uploads/2018/03/PCI-compliance-in-containers-3.gif
 [6]: /wp-content/uploads/2018/03/PCI-compliance-in-containers-4.png
 [7]: /wp-content/uploads/2018/03/PCI-compliance-in-containers-5.png
 [8]: https://www.brighttalk.com/mybrighttalk/channel/16287/webcast/308225/brighttalkhd
 [9]: https://go.sysdigrp2rs.wpengine.com/docker-security-demo