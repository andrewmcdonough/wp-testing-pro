---
ID: 2059
post_title: 'Presenting the New &#038; Improved Sysdig Cloud UI'
author: Marcus Sarmento
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/presenting-the-new-improved-sysdig-cloud-ui/
published: true
post_date: 2015-09-22 07:00:29
---
Here at Sysdig, we are constantly trying to improve our <a href="http://www.sysdig.org/" target="_blank">open source</a> and <a href="https://sysdigrp2rs.wpengine.com/" target="_blank">commercial</a> offerings based on feedback from our users. And over the past year or so we’ve received a lot of very valuable feedback, thanks Sysdig fans! Today I’m happy to announce that we’ve released a significant update to the Sysdig Cloud user interface. We’ve built a brand new user experience, centered around the most common ways you actually use Sysdig Cloud. Read on to learn more about some of the things we’ve done to make using Sysdig Cloud easier than ever before. 

## Explore 

We’ve made the Explore section of our product much easier to use and navigate. You’ll notice that, by default, you have a larger workspace to dig into your infrastructure and analyze different views for your servers and AWS Services. Seeing your whole infrastructure, logically grouped by the tags of your choosing, all in one screen, is a great way to drill down into complex applications and highlight bottlenecks. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/09/explore-ui-blog.png" alt="explore ui blog" width="1500" height="838" class="aligncenter size-full wp-image-2060" />][1] 

Then when you select an entity (a group, host, or container for example) a panel pops up that shows you all the different built-in views (which are pre-configured dashboards for all your common technologies) and metrics available to you. At this point you can easily filter the views and metrics list by simply typing in your topic or keyword of choice. Pivoting between dozens of different views and metrics is now super simple and straightforward. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/09/explore2-ui-blog.png" alt="explore2 ui blog" width="1500" height="838" class="aligncenter size-full wp-image-2061" />][2] 

## Dashboards 

Now, when you view your dashboards, you’ll notice a few improvements that make managing and editing your dashboards that much easier. To start, you’ll notice that if you edit an existing dashboard we make it very clear exactly which values you can edit and what your different options are. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/09/dashboard-ui-blog.png" alt="dashboard ui blog" width="1500" height="838" class="aligncenter size-full wp-image-2062" />][3] 

We also allow you to maximize the entire screen and eliminate the top navigation (if you wanted to display a dashboard in a NOC for example): 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/09/dashboard2-ui-blog.png" alt="dashboard2 ui blog" width="1500" height="839" class="aligncenter size-full wp-image-2063" />][4] 

You can even go full screen with a particular widget if you had one that was particularly interesting: 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/09/dashboard3-ui-blog.png" alt="dashboard3 ui blog" width="1500" height="834" class="aligncenter size-full wp-image-2064" />][5] 

## Alerting 

I’ve already written about our awesome, <a href="https://sysdigrp2rs.wpengine.com/alerting-todays-tomorrows-distributed-containerized-environments/" target="_blank">container-native alerting</a> in the past so I won’t go back into a detailed explanation of our capabilities. Suffice it to say you can alert on any metric we are collecting, and alter the scope so the alert only fires when you want it to. We also have automatic baselines and host comparison alerts where our system can learn what is normal, expected behavior and notify you when we observe a value that is different than what we expect. Pretty powerful stuff. 

With our new user interface, we’ve added two helpful data points to each alert - a **State** and a **Resolved** status. 

If the alert fired from a temporary spike and then went back below the threshold, we tag those notifications as an OK state. However, if the alert fired and the value our system is observing is **still**, **currently** outside of the threshold you’ve set, we tag those notifications as an Active state. This is a great way to quickly identify and surface the issues that are causing performance issues **right now**. 

The Resolved status lets you easily mark particular notifications as resolved. By default we filter out those resolved notifications so you can focus on the notifications that matter - the unresolved ones :) 

Not only do we append these two statuses to every notification, but we allow you to filter the entire notifications view by State, Resolved, and Severity so you have an easy and intuitive way to get to the information you want as fast as possible. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/09/alerting-ui-blog.png" alt="alerting ui blog" width="1500" height="839" class="aligncenter size-full wp-image-2065" />][6] 

## Conclusion 

If you are already a Sysdig Cloud user, you’ll see the new user interface the next time you log in. Thanks to those of you who have provided the feedback that led to this new and improved experience. If you aren’t already a Sysdig Cloud user, I encourage you to <a href="https://sysdigrp2rs.wpengine.com/landing-page/?utm_source=web&utm_medium=blog&utm_campaign=newuiblog092215" target="_blank">sign up</a> so you can try out the new UI yourself and let us know what you think.

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/09/explore-ui-blog.png
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/09/explore2-ui-blog.png
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/09/dashboard-ui-blog.png
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/09/dashboard2-ui-blog.png
 [5]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/09/dashboard3-ui-blog.png
 [6]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/09/alerting-ui-blog.png