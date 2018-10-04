---
ID: 7102
post_title: Joining the Sysdig family.
author: Suresh Vasudevan
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-announces-new-ceo/
published: true
post_date: 2018-04-23 13:52:54
---
I am deeply honored and excited to become part of the Sysdig family. Leading Nimble Storage from its youngest days through an IPO, and then the acquisition by HPE, has been the most gratifying professional experience of my life. My goal was to find something that would match or exceed that, and I believe I have found it at Sysdig!

Over the last year, as I met with dozens of really interesting companies, I had three fairly simple and straightforward assessment criteria:

1.  **Is there a big market that is experiencing rapid and chaotic changes**? 

2.  **Is there a fundamentally new and valuable insight that underpins the company’s technology approach**? 

3.  **Does the founding team have big aspirations and grit**? 

# Rapidly evolving market presents new challenges {#rapidlyevolvingmarketpresentsnewchallenges}

Enterprises are transforming their application development to be far more nimble and cloud-ready by adopting microservices and containers. This is a huge advantage for them, but also presents new operational hurdles as DevOps and DevSecOps move their reachitected applications into production. To illustrate these challenges, let me summarize what I heard from just three out of the hundreds of Sysdig customers:

1.  **Global top 5 Bank**. This bank, with thousands of developers, is moving rapidly towards microservices architectures for their application development, prompted by enormous benefits in terms of time to market and cost of application development. This poses daunting challenges for monitoring and security. How do developers monitor their applications that are dynamically orchestrated across a massive infrastructure spanning over 100,000 hosts? How can run-time monitoring be granular enough to monitor ephemeral, short-lived processes? How can you scale to ingest, store and analyze millions of metrics per second? The list of challenges goes on and on, and necessitated a fundamentally new way to monitor and secure their global infrastructure. 

2.  **A government agency charged with border security, counter-terrorism and fighting crime**. The central IT team has built a centralized "container as a service" platform for all new applications. As the physical infrastructure became much more fluid and dynamic, the IT team needed metrics to diagnose application problems without having to build metrics into each and every application. Further, security was a major concern, and they wanted a container-native solution with deep visibility into containers which were otherwise opaque to their traditional tools. For instance, they were concerned about security breaches and data exfiltration that might go completely undetected by traditional tools because of the short-lived and ephemeral nature of containers.

3.  **A global top 5 Multi-Service Operator (cable / satellite / pay TV)**. This global operator set out to deliver their vast array of video assets as IP delivered web services. They decided to migrate to a container-based microservices architecture to avoid configuration drift (across data centers, OS versions, etc.), to scale on demand, and to streamline handoff between development and production. Given their plan to instantiate these revenue-critical services across data centers and franchisees, they needed a robust monitoring solution that had deep, native integration with Kubernetes without custom instrumentation, and they needed the solution to scale-up and scale-down automatically based on the overall deployment context.

The move to containers and microservices is driven by one key advantage among many - time to market. At the same time, it poses new challenges for monitoring and security, as monolithic applications break down into composable applications comprised of dozens and hundreds of inter-dependent services. Further, the infrastructure underpinning these services will be orchestrated dynamically and will span on-premise as well as public clouds. This massive shift requires rethinking all of the traditional monitoring and security approaches and solutions currently in use.

# Our technology approach driven by key insights {#ourtechnologyapproachdrivenbykeyinsights}

Every one of the customers mentioned above became a Sysdig customer, managing tens of thousands of containers, after an exhaustive evaluation of alternatives, including the alternative of building home-grown solutions. A key reason they selected us is because of the unique approach we have pioneered in our <a href="https://sysdigrp2rs.wpengine.com/product/how-it-works/" target="_blank">Container Intelligence Platform</a>.

Our founder, Loris Degioanni, was the co-creator of the popular open-source network analyzer <a href="https://www.wireshark.org/" target="_blank">Wireshark</a>. In early 2012 he started getting concerned about the ability to derive adequate insights by focusing on network data as a source of truth as IT infrastructure and applications were moving to the cloud. Logs are another source of truth for IT organizations and are very useful, but are often difficult to rely on for run-time monitoring and troubleshooting, particularly for infrastructure monitoring and security in dynamically orchestrated microservices architectures. 

Dwelling on these shortcomings led to the insight that became the inspiration for Sysdig. *The insight was that every interaction with a host - application accesses, network activity, file reads and writes and so on - translates into a system call within the kernel. If one could find a way to enrich system calls with all the relevant context, and translate those into a stream of metrics that can be analyzed real-time and at scale, you would have a third source of truth - at least as powerful, and complementary, to logs and network traffic*.

Further to the core idea that system calls, enriched with appropriate context, could be a foundation for monitoring, securing and troubleshooting microservices, there were a few other key principles that underpin our platform:

1.  *Zero-touch instrumentation*. The platform should be extract information without explicit coding or integration with applications and infrastructure.

2.  *Visibility into containers*: Monitor and secure containers, AND what is inside the containers, including the specific processes and applications that are running within containers. 

3.  *Out-of-box service context*: It is not enough to focus on individual containers, but the platform should map the physical infrastructure and containers into a services view by automatically extracting metadata from orchestrators.

4.  *Scale and dynamic scope*: The metric store needs to be able to handle millions of performance metrics and security events being ingested, stored and analyzed every minute. Perhaps even more importantly, the underlying metric store needs to be flexible enough to allow for hitherto unknown segmentations and analytics on the underlying data.

5.  *Tailored to personas and roles*. The platform should leverage the richness of the underlying data but it should be possible to tailor the output and interactions to different roles - developers, infrastructure managers, security practitioners, forensic experts and so on. 

# A team aspiring to make a difference {#ateamaspiringtomakeadifference}

It seems self-serving to talk about how great the team at Sysdig is, which it is :) 

I have had the fortune of working at companies such as NetApp and Nimble, and have come to believe deeply that organizational culture is perhaps a more enduring differentiator than strategy or execution. The team at Sysdig has all the attributes that are key ingredients to a great culture. The team is proud of what we have built and yet humble in recognizing that we are just starting our journey in many ways. We are thrilled to have a community of over a million downloads of <a href="https://www.sysdigrp2rs.wpengine.com/opensource" target="_blank">opensource Sysdig and Sysdig Falco</a>, and hundreds of Enterprises deploying our commercial products. The team is resolute in being focused on the long-term mission of being a thought leader and driving the industry transformation towards containers and microservices as the foundation for modern application development. We are pretty certain that we will have tons of fun along the way.

How could I not be excited to be part of this journey?