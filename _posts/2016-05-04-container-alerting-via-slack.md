---
ID: 2589
post_title: Container alerting via Slack
author: Apurva Dave
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/container-alerting-via-slack/
published: true
post_date: 2016-05-04 17:11:10
---
We love Slack and I know many of you do too. So now you can monitor containers (and your apps!) without ever leaving Slack, using our really easy-to-use integration for alerts from Sysdig Cloud.

The integration only requires you to authenticate your Slack domain and the channel you’d like to send alerts to. You can find this in settings -> notifications.

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/05/Slack1.png"></a>[![sysdig and slack][1]][1] 

Then, choose Slack as an action to take when an alert is fired. Note that you can select as many options as you’d like… get a slack message, get an email, and more interestingly [capture a system trace of every request on the offending container or host during an alert.][2]

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/05/Slack2.png"></a>[![sysdig and slack][3]][3] 

Alerts appear in Slack well formatted, and most importantly with a link back to Sysdig Cloud that allows you to see the entire state of your system when the alert fired. The result? Simpler, faster container monitoring and alerting.

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/05/Slack3.png"></a>[![sysdig and slack][4]][4] 

The Slack integration is already live in our product - give it a shot and tell us that you think!

Oh, and speaking of Slack, please [join our public Slack channel][5] to talk about our open source sysdig technology!

 [1]: /wp-content/uploads/2016/05/Slack1.png
 [2]: https://sysdigrp2rs.wpengine.com/blog/troubleshooting-containers-when-theyre-gone/
 [3]: /wp-content/uploads/2016/05/Slack2.png
 [4]: /wp-content/uploads/2016/05/Slack3.png
 [5]: https://slack.sysdigrp2rs.wpengine.com/