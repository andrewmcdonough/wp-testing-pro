---
ID: 322
post_title: The Operations Identity Crisis
author: Chris Crane
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/operations-identity-crisis/
published: true
post_date: 2014-09-21 10:00:24
---
I was recently editing another of our blog posts and found myself typing the phrase “...developers and operations people.” What an awkward description - “operations people.” The linguistically obvious “operators” isn’t right either though. And the more I struggled to come up with something better, the more I realized something that has been nagging me for a long time: there is no concise, universally agreed upon title for “operations people.”

## The Data

A quick piece of analysis hammered this point home for me. As we’ve been collecting beta signups for Sysdig Cloud, a portion of people who request invites have been kind enough to share their job title with us. Here’s the breakdown by category across hundreds of those job titles (with some examples illustrating the type of title in each category):

[<img class="aligncenter wp-image-327 size-full" src="/wp-content/uploads/2015/08/image01-e1408471165440.png" alt="image01" width="989" height="540" />][1] 
Maybe it's just me, but I found this fascinating. While devs are essentially all Engineers with a few Architects sprinkled in*, operations titles are scattered all over the map. Sure there are plenty of SysAdmins, but there are also Systems Engineers, Operations Engineers, DevOps Admins, Infrastructure Managers, IT admins, and dozens of others. Some of the titles do describe more specific roles, such as Network Admin, but most of these are just vague permutations of the same general theme. Most of these operations-focused people are doing very similar jobs with very similar skills and responsibilities. So why so many different ways of labeling the same role?

I’d like to argue that many of the factors driving this proliferation of titles are actually the same factors behind why we think the world needs a new generation of performance management tooling. Let me explain.

## The Times They Are a-Changin’

Here at Sysdig, we see a profound shift happening in the architecture of the internet, unlike anything seen in the decades since the internet first developed into its modern form. Applications have become massively scaled, distributed and service-oriented, and in order to support these modern applications, the infrastructure itself has become programmable. We’re transitioning from a fundamentally hardware-based infrastructure to a fundamentally software-based infrastructure.

Of course, these modern, scaled, distributed, virtualized systems are stunningly complex and difficult to administer. A new slew of processes and methodologies and philosophies have evolved to help IT teams deliver applications effectively. Agile development, test driven development, continuous integration, continuous delivery, continuous deployment and of course the infamous [DevOps][2], to name a few. And correspondingly, a new slew of technologies have been invented to enable and simplify these processes - technologies like automated regression testing, deployment management, auto-scaling and load balancing, and configuration management. A reasonably sized team can now develop, deliver and administer a modern, web-scale application, and it’s only because all of these new technologies and methodologies make it possible.

But, and here’s the key point, these technologies and methodologies also require a *completely new set of skills*, especially for operations. Development may be based on new languages and processes; but fundamentally, the core skill remains the same: write code. But operations has gone from plugging in servers in a data center and installing Linux upgrades to scripting Chef recipes and architecting Docker-based environments. That’s quite a shift.

Now it’s worth noting, many of these advancements are being driven by developers who have seen the value in taking some ownership themselves over operations-type responsibilities. But in our experience, the vast majority of practitioners of modern operations are coming from the traditional operations backgrounds: system admins, network admins, DB admins, Linux admins, etc.

And the other interesting thing that’s happening here is that on top of the creation of all these new skillsets, we’re also losing old ones. Who needs a network admin, when Amazon owns your network in a black box? Now don’t get me wrong, a working knowledge of network administration is still extremely useful, even in cloud-based infrastructures. But these skills are becoming less critical as a full-time job. In a similar trend, many of these traditional areas of focus are being lumped together into single roles. Similar to the full-stack engineer, we’re now getting full-stack operations.

## Back To The Identity Crisis

Given all this, I think it’s no surprise that operations engineers are searching for new ways to label their roles. Check out the picture on the [Systems Administrator wikipedia page][3]**:

[<img class="aligncenter wp-image-324 size-full" src="/wp-content/uploads/2014/08/Screen-Shot-2014-08-19-at-10.48.48-AM.png" alt="Screen Shot 2014-08-19 at 10.48.48 AM" width="428" height="562" />][4] 
How many Systems Administrators today would identify with this image? It would seem “SysAdmin” might not be projecting the intended message. So what do you call yourself if you want to convey to colleagues what you actually do, or to potential hirers what your skillset actually is, or to potential candidates what the role you're hiring for actually is? Operations Engineer certainly seems to be a popular option, but there are many others. Pages upon pages have been written on the pro’s and con’s of putting DevOps in your title. Regardless of where you fall in the debate, it’s clear that at least some of the intention of a DevOps title is simply to convey a modern skillset that doesn’t seem to have an agreed upon name.

## Conclusion

So where does this leave us? I don’t have the right answer, and I’m certainly not trying to start a movement behind any one title. But I do think that this title proliferation in the world of operations is a fascinating, frustrating, and unnecessary phenomenon, and I think it’s a symptom of the fundamental changes happening within the industry. And I will say this: unless the wisdom of the crowd is able to settle in some kind of consensus on the proper name for this new skillset - this ability to administer modern, distributed, programmable, massively scaled systems - until we agree on what to call that, I don’t think this issue is going anywhere.

## Epilogue

I mentioned above that many of the factors driving this identity crisis are the same factors inspiring our team to create Sysdig and Sysdig Cloud. I’ll get into this much more in a later blog post, but the crux of it is this: these amazing new technologies and processes that make administering a modern web application possible also do something else. They exacerbate all of the traditional challenges associated with performance management. Monitoring and alerting, finding visibility and context, troubleshooting for root cause - all these get exponentially harder when you’re dealing with a massively scaled, remarkably transient, distributed, virtualized infrastructure. Here at Sysdig we’re building a new generation of performance management tools to help solve these problems for the new generation of “operations people”.

And by the way, if this sounds interesting to you, [we’re hiring][5].



Thanks!

  

**I'm not sure why no "Developers" showed up in this analysis.. perhaps Engineer is seen as the more professional sounding title? A question for another day.*



***No offense intended to you sir, I'm sure you're a great guy, and there's a beer on the house waiting for you at Sysdig HQ.*

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/08/image01-e1408471165440.png
 [2]: https://www.youtube.com/watch?v=bW7Op86ox9g
 [3]: http://en.wikipedia.org/wiki/System_administrator
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2014/08/Screen-Shot-2014-08-19-at-10.48.48-AM.png
 [5]: https://sysdigrp2rs.wpengine.com/jobs