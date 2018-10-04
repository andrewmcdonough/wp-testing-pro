---
ID: 2411
post_title: >
  How ShareThis uses Kubernetes in
  production
author: Apurva Dave
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/how-sharethis-uses-kubernetes-in-production/
published: true
post_date: 2016-02-16 10:04:17
---
Right now, there is a lot of talk about using containers in production and very few cold, hard case studies about the good, the bad, and the real business value of it.

Juan Valencia, Technical Lead at ShareThis, wrote an in-depth and insightful blog post last week on [kubernetes.io][1] that shows the progression of a development organization from skepticism, to promise, to a productive reality. FYI, ShareThis is a service that helps website publishers drive engagement and consumer sharing behavior across social networks.  
  
A few highlights: 

<p style="padding-left: 30px;">
  “Our Kubernetes cluster currently processes 800 million requests per day with the plan to process over 2 billion requests per day in the coming months.” <br /> <br /> “After 1 month, our organization went from having a few DevOps folks, to having every engineer capable of modifying our architecture.” <br /> <br /> “After 4 months, [we were deploying] 40 apps per week. Only 30% of our apps have been migrated, yet the gains are not only remarkable, they are astounding.”
</p>

  
I highly encourage you to give [Juan’s post][1] a read. I think it would stir up a good conversation in any organization thinking about making the leap to a cloud-native architecture. ShareThis also happens to be a Sysdig Cloud customer, and we’re excited to be a small part of their next generation infrastructure. If you’d like to learn more about troubleshooting Kubernetes in production, check out [this post from our engineering team.][2]

 [1]: http://blog.kubernetes.io/2016/02/sharethis-kubernetes-in-production.html
 [2]: https://sysdigrp2rs.wpengine.com/a-sysdigkubernetes-adventure-part-2-troubleshooting-kubernetes-services/