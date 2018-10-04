---
ID: 2558
post_title: 'Q&#038;A with Loris Degioanni: On raising $15m and democratizing containers'
author: Apurva Dave
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-raises-15m/
published: true
post_date: 2016-04-20 17:17:06
---
Cha-ching! Sysdig has put a little more money in the bank. So what’s the story? I decided it would be fun to put Loris under the spotlight and see what he has to say.

**Apurva:** Loris! Congrats. First, can you provide the basics around the funding?

**Loris:** Thanks! 

We raised [$15 Million and it’s our Series B][1]. Accel and Bain Capital Ventures co-led the round. Bain had invested in the seed round, and Accel had led our A Round. It’s pretty telling that our existing investors wanted to put more money in. We really like working with them too, so that made the process pretty easy.

We’re really excited about the next phase of the company, which is exactly what this funding will support.

**Apurva:** How does the funding relate to the state of the business and the product?

Loris: I think the funding is a statement about two different things: the state of the container market and the state of our product/business. On the market, while containers are still in the early stages of adoption, the market is picking up fast and has massive potential. When you read the news you see everyone from small software companies to large banks like [Goldman moving to containers][2]. 

In a short period of time, our open source project has hundreds of thousands of downloads, and a good chunk of that is due to our unparalleled container visibility. In just months since the GA of our first commercial product, we’ve already surpassed 100 enterprise customers monitoring their container environments. That’s really strong evidence of need, and this funding is also evidence of that.

**Apurva:** OK, so what are you going to do with the money?

**Loris:** Well, throw a big party in Vegas of course! Kidding, kidding, a party in vegas isn’t nearly as much fun as coding cool projects. Is it ok to admit that??

Honestly Sysdig is growing on all fronts right now. We’re going to invest in engineering to deliver more functionality, and our go-to-market teams to make sure we’re doing our best to support customers and design the additional functionality they need.

We also want to continue to invest in open source. My experience (first with Wireshark and now Sysdig) has always been rooted in the open source community, and I want to continue to give back. 

**Apurva:** Perfect segway. Talk to us about the product functionality you’re creating. 

Sysdig Cloud is really in a fun place of its evolution for product development. The core of our platform for monitoring containers is in place, so we’re developing highly valuable features at a very rapid rate. Just take a look at [everything we released over the past few months][3]! Customers shouldn’t have to use chewing gum and bailing wire with their old solutions that don’t understand containers. Here’s how I’d sum up our plans for container monitoring:

*   **See more with less effort:** We have a big focus on passive instrumentation - seeing more without requiring the user to do more. With our full-stack view of the environment: containers, apps inside containers, underlying infrastructure and clouds we can help people monitor the user experience that they are delivering, not just whether a container is using too many resources. 

*   **Ecosystem integration:** Customers love us because we don’t treat the container ecosystem integration as a checkbox. We built native integrations with orchestration systems like Kubernetes and Mesos, which allow our users to see the real-time state and distribution of their containers in a more effective way. Over time we’re going to go even deeper with container management software and clouds.

*   **Monitor containers and microservices:** The shift to microservices is coming - it’s the natural evolution of container adoption. Microservices will fundamentally change how devops teams manage the operations of their software, but the tooling isn’t yet there to support these new workflows. It of course all starts from really understanding the container, which is what we do today.

**Apurva:** And what about on the open source front? What’s the vision there?

**Loris:** Oh, be careful, don’t get me started. We’ve got a lot of great ideas to explore here. Our core instrumentation is so versatile and flexible that there are many directions we can go in.

We’re in fact going to release a new project soon, but I’ll resist the urge to tell you about it until the beta is released! I’ll instead say we have a philosophy: containers and their underlying operating systems are really complex, but the tooling that operates them doesn’t need to be. We want to democratize access to information that has previously only been in the hands of linux gurus. It’s a big undertaking, and that’s why I think a community-based approach is the only way to do it.

**Apurva:** You’re keeping the audience on the edge of their seats waiting for the next announcement. I like that. Anything else you’d like to add?

**Loris:** I’d just like to say thank you to all the people and companies that have worked with us since the launch of the Sysdig. We couldn’t have done this without a ton of support, advice, and most of all patience! Thanks for believing in us, we’ll continue to make it worth your while! And finally thanks to our rockstar team… we’re hiring across all fronts, [please come and join us][4] if you have a passion for containers and open source!

**
If you have any questions for Loris about Sysdig stop by our AMA tomorrow on the [Sysdig Slack Channel!][5]**

 [1]: https://sysdigrp2rs.wpengine.com/press-releases/sysdig-series-b-container-monitoring/
 [2]: http://blogs.wsj.com/cio/2016/02/24/big-changes-in-goldmans-software-emerge-from-small-containers/
 [3]: https://sysdigrp2rs.wpengine.com/blog/sysdig-cloud-spring-2016-release/
 [4]: https://sysdigrp2rs.wpengine.com/jobs/
 [5]: https://slack.sysdigrp2rs.wpengine.com/