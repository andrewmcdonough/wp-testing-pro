---
ID: 2494
post_title: 'Deploying a private PaaS: The good, the meh, and the aw crap'
author: Cody Boggs
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/deploying-private-paas-good-meh-aw-crap/
published: true
post_date: 2016-03-28 22:50:00
---
We occasionally run across awesome posts that we want to share with you. This one is about building your own PaaS with containers from our colleague [@strofcon][1]. We're happy to provide it here; it was originally published on his own [blog][2].

Moving your organization's dev and prod deployment architecture to a [PaaS][3] model can bring a lot of benefits, and a good chunk of these benefits can be realized very quickly by leaning on public PaaS providers such as RedHat's [OpenShift Online][4], Amazon's [ECS][5], or Google's [Container Engine][5]. Sometimes though, a public PaaS offering isn't a viable option, possibly due to technical limitations, security concerns, or enterprise policies.

Enter the private PaaS! There are a number of options here, all of which offer varying degrees of feature abundance, technical capability, baked-in security goodies, operational viability, and other core considerations. As with anything in the tech world, the only clear winner among the various tools is the one that best fits your environment and your needs. No matter what you choose, however, there are some key things to consider during evaluation and implementation that span the PaaS field independent of any particular tool.

Let's walk through some of the ups and down of deploying a private PaaS. Please keep in mind that this post isn't about "Private PaaS vs. Public PaaS". Instead it assumes you're in a situation where public cloud offerings aren't a viable option, so it's more about "Private PaaS vs. No PaaS."

### The Good

A full listing of all the good things a PaaS can bring would be pretty lengthy, so I'll focus on what I think are the most high-value benefits compared to a "legacy" deployment paradigm. 

**Increased Plasticity and Reduced Mean Time to Recovery**



Plasticity: the degree to which workloads can be moved around a pool of physical resources with minimal elapsed time and resource consumption



Mean Time to Recovery (MTTR): the time taken to recover from a failure, particularly failure of hardware resources or bad configurations resulting in a set of inoperable hosts (at least in the context of this post)



Legacy architectures typically see one application instance (sometimes more, but it's rare) residing on a single (generally static) host. Even with some manner of private IaaS in place, you're still deploying each app instance to a full-blown VM running its own OS. This makes it difficult to attain any reasonable plasticity in your production environments due to the time and resources needed to migrate a compute workload, which in turn forces MTTR upward as you scale up.



PaaS workloads generally eschew virtualization in favor of containerization, which can dramatically reduce the time and resources needed to spin up and tear down application instances. This allows for greatly increased plasticity, which consequently drags MTTR down and helps keep it reasonably low as you scale up.



**Reduced Configuration Management Surface Area**



Configuration management is not only sufficient for handling large swaths of your core infrastructure, but has effectively become necessary for such. That said, there's a lot of value in reducing the surface area touched by whatever config management tool(s) you're using. A particularly unhealthy pattern that some organizations find themselves following is one in which every application instance host (virtual or not) is "config managed." In the case of bare metal hosting this makes good sense, but in the event that you're deploying to VMs... it's no good. At all.



Reducing this surface area can act as a major painkiller by requiring much simpler and more consistently applied configuration management... er... configurations. With a PaaS, you only need to automate configuration for the hosts that run the PaaS (and hosts that run non-PaaS-worthy workloads), not the individual application instance containers. This makes life suck a lot less.



**Consistency and Control**



No one likes an overzealous gatekeeper - they're generally considered antithetical to continuous integration and deployment. On the other hand, an environment that isn't in a position to deploy to a public cloud platform is also very unlikely to be in a position to live without some fairly high level of control over its code deployments. The key here is automated gate-keeping, and a PaaS gives you a decent path to accomplishing this.



Running a private PaaS for both staging and production environments gives you ample opportunity to funnel all code deploys through a consistent pipeline that has sufficient (and ideally minimal-friction) automated controls in place to protect production. This allows Infrastructure Engineers to provide a well-defined and mutually-acceptable contract for getting things into the staging PaaS, and consequently provides a consistent and high-confidence path to production - all without necessitating manual intervention by us pesky humans. Basically, if the developer's code and enveloping container are up to snuff by the staging environment's standards, they're clear to push on to production.



Regarding consistency, developer's also gain confidence in their ability to deploy services into an existing ecosystem by being able to rely on a staging environment that mirrors production with far fewer variables than might otherwise have been present in a legacy architecture. Dependencies should all be continually deployed to staging, and thus assurance of API contracts should be much closer to trivial than not.



**Making DevOps a Real Thing**



Everyone's throwing around DevOps these days and it's exhausting, I know - but I'm still going to shamelessly throw my definition on the pile.



My current take on DevOps is that an engineering organization most likely contains ops folks who are brilliant within the realm of infrastructure, and developer folks who are brilliant within the realm of application code. If you consider a Venn diagram with two circles - one for infrastructure and one for code - most organizations are likely to see those circles sitting miles apart from one another, or overlapped so heavily as to be indecent. The former diagram could be called "the DevOps desert," and the latter "choked with DevOps". Neither of these are particularly attractive to me.



A well-devised and even-better-implemented PaaS has the potential to adjust that diagram such that the two circles overlap, but only narrowly. Ops people focus hard on infrastructure, and dev people focus hard on code, with a narrow and remarkably well-defined interface between the two disciplines. There's still plenty of room for collaboration and mutually-beneficial contracts, but dev doesn't have to muddy their waters with infrastructure concerns, and ops doesn't have to muddy their waters with code concerns. I think that could be a beautiful thing.



### The Meh



There aren't a ton of objectively "bad" things about running a PaaS vs. a legacy architecture, but there are a few necessary unpleasantries to consider. 

**Introducing Additional Complexity**



A PaaS is an inherently complex beast. There are mitigating factors here and the complexity is generally worth it, but it's additional complexity all the same. Adding moving parts to any system is, by default, a risky move. But in today's market and at today's scale, complexity is a manageable necessity - provided that you choose your PaaS wisely and hire truly intelligent engineers over those who can throw out the most buzzwords.



**Network configurations**



Containers need to be able to route to one another across hosts in addition to being able to reach (and be reachable by) other off-PaaS systems. Most PaaS products handle this or provide good patterns for it, but it still introduces a new layer of networking to consider during troubleshooting, automation, and optimization.



**Maintenance and Upgrades**



If you've build your PaaS and its integration points well, you'll end up with a compelling deployment target on which lots and lots of folks are going to run their code. This can make it tricky to handle host patching and upgrades without impacting service availability. That staging environment (and maybe even a pre-staging environment solely used for infrastructure automation tests) becomes very important here.



**Data Stores**



Anything that needs local persistent storage (think databases and durable message buses) are unlikely to be good candidates for PaaS workloads. It can be made to work in some cases, but unless you're a big fan of your SAN, you're likely best to keep these kinds of things off the PaaS. Even if you can make it work, I'm not yet convinced that there is much value in having such a workload reside in such a fluid environment.



**Capacity Planning and Resource Limits**



Container explosions can bite you very quickly, most likely due to a PaaS' self-healing mechanism glitching out. This is a complex component and you're guaranteed to find really... um... "special"... ways of blowing out your resource capacity until you get a good pattern figured out. A clear pattern for determining and enforcing resource limits is going to be incredibly helpful.



Capacity planning also demands relevant data, which means you'll need some solid metrics, and that brings us to...



### The "Aw, Crap..."



There are a couple of problems that a PaaS introduces which will, at some point, demand your attention - and punish you harshly should you neglect them.



**Single-Point-of-Failure Avoidance**



SPoFs are anathema to high availability. If you're going to introduce something like a PaaS, you would do well to think very hard about every piece of the system and how its sudden absence might impact the larger platform. Is the cluster management API highly available? Then you'll need to stick a load balancer in front of those nodes. What happens if the LB goes down though?



Or what if you're running your PaaS nodes as VMs? That's fine, until you factor in hypervisor affinities - can't afford to have too many PaaS nodes residing on a single hypervisor. Even if you account for node -> hypervisor affinities, can you ensure that your PaaS won't shovel a large portion of a service's containers onto a small subset of nodes that do reside on the same hypervisor? The extra layer of abstraction away from bare metal here is likely to introduce new SLA-impacting failure scenarios that you may not have considered.



You're very likely to be able to mitigate any SPoF you come across, but they're likely to be slightly different in some cases than what you've handled before, and it's worth considering how to avoid them up front.



**Metrics and Monitoring**



Disclaimer: I obsess over metrics. It's almost unhealthy, really, but hey - it works for me. So it's not too surprising that introducing a PaaS makes me **very** curious about how one can go about gathering relevant metrics from the various layers involved. There are effectively three layers or classes of metrics you need to concern yourself with in a typical PaaS: host, container, and application.



Host metrics are just as easy as they've ever been, so that's not of much concern. Collection and storage of metrics for semi-static hosts is largely a solved problem.



Container metrics (things like CPU, memory, network, etc. used per container) isn't too terribly difficult to collect, though storage and effective visualization can be more difficult due to the transient nature of containers - particularly when those containers are orchestrated across multiple hosts.



Application metrics (metrics exposed by the actual application process within each container) are potentially a real bear of a problem. Polling metrics out of each instance from a central collection tool isn't too attractive since it's fairly difficult to track transient instances such as those that reside in a PaaS. On the flip side, having each application instance emit metrics to some central collector or proxy is feasible, but storage and visualization are still difficult since you'll inevitably need to see those metrics in the context of orchestration metadata that is unlikely to be available to your application process.



These are not entirely insurmountable problems, but there are definitely not many viable products currently available that have a good grasp on how best to present the data generated within a PaaS. In fact, so far the only "out of the box" solution I've found so far that handles this well is Sysdig Cloud. These folks are onto something pretty awesome, and I plan to elaborate on that in my next post.



**Most People Are Doing Containers All Wrong**



Containers should run a single process wherever possible, and should be treated as entirely immutable application instances. There should be no need to log into a container and run commands once it's running on the production PaaS. Images should be built layer upon layer, where each layer has been "blessed" by the security and compliance gods. Ignoring these principles is worthy of severe punishment in the PaaS world, and your PaaS will be all too happy to dish out said punishment. Don't go in unarmed.



### Summary



Implementing a PaaS in your environment is most likely well worth the effort and risk involved, just so that your organization can push forward into a more scalable and modern architecture. While there are certainly a few potential "gotchas," there's a lot to be gained by going into this kind of project with a level head and enough information to make wise decisions. Just keep in mind that this landscape is changing pretty rapidly, and moving to a PaaS is certainly not a decision to be made lightly - but it's still probably the right decision.

 [1]: https://twitter.com/strofcon
 [2]: http://tech.strofcon.org/2016/03/deploying-private-paas-good-meh-and-aw.html
 [3]: https://en.wikipedia.org/wiki/Platform_as_a_service
 [4]: https://www.openshift.com/features/
 [5]: https://aws.amazon.com/ecs/