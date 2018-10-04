---
ID: 3446
post_title: 'On monoliths, Kubernetes, and monitoring: Transitioning to Docker at Major League Soccer'
author: Apurva Dave
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/monoliths-kubernetes-monitoring-transitioning-docker-major-league-soccer/
published: true
post_date: 2016-12-14 23:11:27
---
Earlier this week at [Tectonic Summit][1] hosted by CoreOS, we heard Brian Aznar speak about his experience migrating to Docker and Kubernetes, and how his monitoring strategy changed as well. Brian is the director of engineering for Major League Soccer. Brian was interviewed by Loris Degioanni, founder of Sysdig. 

Below is the text of the conversation, slightly edited for clarity. 

\------------------------

****

**Loris Degioanni (LD):** Brian tell us about your role. What do you personally do? Are you the front-line techie or the stakeholder?

**** 

**Brian Aznar (BA):** I’m the director of engineering at Major League Soccer. I’m in charge of engineering for web, mobile, and content management systems; I run operations as well. I spend a lot of my time on hiring, how we build the team, and strategic decisions about where we need to go with our platforms and tooling. I'm working with my eng teams to make decisions that I feel put the business on a path for sustainable competitive advantage. In essence, I’m the stakeholder for our strategic technology direction.



To put it another way - I’m the highest level at which anyone really cares that we’re using Kubernetes - and that’s a good thing. The rest of the management team can stay focused on soccer.



\------------------------

****

**LD:** What's the Major League Soccer technical environment like?

**** 

**BA:** We’re a small team and have transitioned to an agile environment. This is the key for our technology decisions, so anything we take on needs to be maximally supportable by everybody. We use Javascript (node, react, etc), and Drupal, among other tools.



The size of our environment has changed considerably: initially we were 75-80 or so EC2 instances in two regions, and after K8s it was about 56. Our actual production workload is handled on 34 instances and a lot of those are beefy AWS c4.xlarges. Our worker cluster size is 10 + 5 running ~ 130 pods. We’re always looking for ways to develop and deploy with greater efficiency and consistency, and Kubernetes enabled us to gain more infrastructure utilization.



\------------------------

****

**LD:** Let's start by talking about your decision to adopt containers and Kubernetes. Can you talk about how that happened?

**** 

**BA:** When I joined MLS, we already had our newer microservices running in AWS with Docker and CoreOS Fleet orchestrating.Our older web platform (Drupal) was running on bare metal servers in our colo. 2015 was about getting the latter into AWS. To get everything else into AWS we had to make decisions around deployment, and figure out how to consolidate our technology stack.



CoreOS and Fleet weren’t particularly scalable and, besides, we saw that CoreOS was pushing Kubernetes. Kubernetes was the final piece of the puzzle in terms of how we currently manage all of our containers now they are all running on a consolidated set of instances. Of course, we had to test it to make sure it was supportable, and we hired a devops lead, John Dodson, to vet and implement our vision. It took six to seven months for us to fully adopt the system.



\------------------------

****

**LD:** Let's tread on some sensitive ground. Rumor has it that you containerized a monolithic application as well?

**** 

**BA:** Yes, we did have a monolithic Drupal app, which we had to decide whether to containerize or not. In the end, we decided to do it in order to maintain consistency and supportability. Prior to this we had disparate build and deployment systems, so going to Docker for our monolith was primarily for consistency. It just meant that we had a cohesive way to deploy for all applications in our technology stack. It was more a support and ops issue than a dev or religious issue.



\------------------------

****

**LD:** Now you've got Kubernetes and Docker in play. So everything works great, all the time right?

**** 

**BA:** Yeah, and work is all rainbows and unicorns now. In terms of higher server utilization, Docker and Kubernetes is the next level of cost optimization. It really gets you thinking about your infrastructure usage in a different way, as you can have fewer beefier servers as opposed to lots of smaller ones. This is one way to scale and deploy more effectively.



There were a couple of challenges when we first moved into the new environment. During setup, we had a couple of minor issues with configuring into AWS, mainly to do with elb naming. John’s got the gory details and has blogged about it.



There’s complex networking that goes on in containerized environments, which can be pretty opaque unless you have a good monitoring strategy in place. Because of the complex networking, I was initially concerned about how that would affect performance and how we would even know where to look and pinpoint everything within Flannel (our overlay network). But these concerns ended up being non-issues.



Having the entire stack on one cluster makes infrastructure performance exploration / monitoring very efficient, and the idea of self-healing applications is actually real for us now.



Gaining container affinity also required a little trial and error, and required good visibility with careful attention to the system. With Kubernetes and Docker in place, we also had to understand the performance profile of our systems and how they’ve changed. For instance, with Drupal there’s a 250ms hit in server performance. We had to ask ourselves: was that okay? Could we profile that and then improve it?



Overall, Kubernetes has taken a lot of what used to be tier 1 issues and made them tier 2. That’s incredibly powerful. Hair-on-fire issues of the past are much, much less common.



\------------------------

****

**LD:** So you touched on monitoring, my favorite topic. Can you talk more about how your monitoring needs changed as you moved to Kubernetes?

**** 

**BA:** Sure. We were using a popular cloud monitoring tool before we moved to Kubernetes, but we adopted Kubernetes as much for its opinionated approach as its technology. We wanted a monitoring system that fully understood and adopted this kind of approach as well. Basically, we were looking for a monitoring tool that had first class Kubernetes support, and Sysdig had that out of the box.



We can look at our microservices more precisely in Sysdig as it uses the logical definitions that Kubernetes provides. In AWS, we can look at the cluster as a whole, zoom in to the namespace, then into the pod/container and back out. This allows for a natural kind of infrastructure and application exploration of performance and resource utilization.



One of Sydig’s killer features is the service response time. For instance, our video ingestion micro services use queues, and with Sysdig we can monitor each microservice's response time, and whether that response time is due to the given service or something upstream from it.



Another big thing is instrumentation. The kernel-level approach that Sysdig uses is pretty powerful. It means we can see everything - without sidecar containers in pods - even when Kubernetes is making decisions to move things around, which is kind of a requirement in dynamic environments.



A great example here is pod affinity, scheduling, and tuning. We are able to see which pods are running very hot in which instances. Because we run beefy nodes with lots of pods, there can be conflicts. I like to call this the ‘wedding planner’ problem--there are always people at a wedding whom you don’t want to sit at the same table. We need to know that conflicts might be happening, see them, and plan around them.



\------------------------

****

**LD:** Let's bring this home Brian. What advice would you give someone who is at the beginning of their Kubernetes journey?

**** 

**BA:** Journey is the right word. First, I’d say think about your objectives. Docker itself is about letting your devs code faster, essentially providing more revenue to the business. Kubernetes is about cutting costs and increasing reliability, so performance may be traded off. You’ll have to balance speed versus reliability in this transition. I have two sets of more detailed advice depending on your position.



If you are already on Docker and in the cloud then the good news is the hard part is already done. Issues can vary depending on your cloud provider as you move to orchestration. For the mosts part K8s works great with AWS, but autoscaling isn’t currently supported aside from custom lambda / cloud watch code, although gce supports it. There may be more gotcha’s like that lurking around that you have to be aware of or stumble across. Also as I mentioned before, the devops team had specific issues with K8s in AWS and elb naming.



If you’re not yet Dockerized and/or in the cloud, then implementation is a more involved task. The sooner you get your application architected properly for the cloud the better. Following the 12 factor app methodology will greatly simplify this process.The best case scenario is that you start out that way, while in the worst case you would need to try and hammer your existing architecture into the shape required, which increases the likelihood of problems. That’s like our experience with Drupal. The real performance concerns with Dockerization seem to be anything to do with with file / volume access. I recommend choosing your networking configuration wisely; we like Flannel.



Next, aim for simplicity in the long term, but expect complexity in the short term. You will need to learn and support new technology, find its gaps, and skill up your team to solve any issues. Finally, visibility becomes even more important when you have multiple layers of virtualization, which will be the case when running K8s with Docker in the cloud. Ultimately, the complexity that I described above demands more insight into your services in a way that philosophically fits with Kubernetes and Docker.

 [1]: https://tectonic.com/summit/