---
ID: 3118
post_title: >
  Improved alerting to up your docker
  monitoring game
author: Apurva Dave
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/improved-alerting-docker-monitoring-game/
published: true
post_date: 2016-10-12 23:48:11
---
Let’s face it, the best dashboard is the one you never have to look at. A good monitoring system should be alerting you to problems that you have, and do it in a way that’s best for your own workflow. To that end we’ve been busy adding a ton of new alerting features. 

Let’s start with a couple we added earlier this summer:

**[Multi-Condition Alerting:][1]** Fine tune your alerts with new functionality. Advanced alerts allow you to define a threshold as a custom Boolean expression which can involve multiple conditions. These advanced expressions require a specific syntax, which we dive into further in this post.

**[Slack Alert Notifications:][1]** We love Slack and I know many of you do too. So now you can monitor containers (and your apps!) without ever leaving Slack, using our really easy-to-use integration for alerts from Sysdig Cloud.

And now we’re bringing you more:

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/10/Alert-Resolved-Status.png"></a>[<img alt="Alert Resolved Status" class="alignnone" height="536" src="/wp-content/uploads/2016/10/Alert-Resolved-Status.png" width="750" />][2]

**Alert Resolved notifications:** This is an oft-requested feature from our customers. Automatically get a resolution notification on the same channel as an alert. You can configure the resolution notice to occur when the condition returns to normal, the alert is manually resolved, or both.

**VictorOps Integration:** Yes! Now you can send your Docker monitoring alerts from Sysdig right to VictorOps. Here are more details direct from their knowledge base.

**WebHook output from Sysdig alerts:** When an alert isn’t enough, you need a webhook. We got that too! Now you can trigger a web post off an alert, and automate solutions to common problems in your infrastructure.

**OpsGenie Integration:** Oh man, this keeps getting better and better. Power your alerting with OpsGenie out-of-the-box.

We’re not done yet!

We’re going to keep improving our alerting capabilities, with plans for a new easier alerting workflow, the ability to alert on events in addition to metrics, and much more. Stay tuned!

 [1]: https://sysdigrp2rs.wpengine.com/blog/multi-condition-alerting/
 [2]: /wp-content/uploads/2016/10/Alert-Resolved-Status.png