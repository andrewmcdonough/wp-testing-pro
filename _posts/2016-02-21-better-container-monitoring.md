---
ID: 2418
post_title: >
  Better container monitoring with the
  number panel
author: Apurva Dave
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/better-container-monitoring/
published: true
post_date: 2016-02-21 21:21:10
---
## A little secret: Container Monitoring loves numbers

...and we love them even more now that we’ve introduced number panels for Sysdig Cloud dashboards and views. Rather than describe them, let’s just take a look as they monitor a container environment: 

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/02/Number-Panel-Blog-1.png"></a>[<img alt="Container Monitoring with Sysdig Number Panels" class="alignnone" height="536" src="/wp-content/uploads/2016/02/Number-Panel-Blog-1.png" width="750" />][1] 

As Sysdig Cloud has grown in importance to our customers, we find that we’re not only being used as a troubleshooting tool, not only as an alerting tool, and not only as a dashboarding tool… but we’re also being used as the “source of truth” running as a wallboard in the office or the NOC where people can quickly glance and get a picture of what’s going on. The number panel works alongside our time series, bar, and table panels to make this all come together. 

How to create a number panel

It’s pretty simple: 

1.  Mouse over a panel
2.  Click the “Settings” gear within that panel
3.  Pull down on “Show As”
4.  Choose “Number”



<a data-rel="lightbox-0" href="/wp-content/uploads/2016/02/number_panel_feature.gif"></a>[<img alt="Container Monitoring with Sysdig Number Panels" class="alignnone" height="536" src="/wp-content/uploads/2016/02/number_panel_feature.gif" width="750" />][2] 

...and you’re done! No PhD in statistics required. Just as an FYI, the number panel is aggregating data over whatever timeframe you’ve set in the dashboard or view. In the screenshot above, it’s a 2-hour window, and in the example below, it’s operating in real-time.

As a bonus, you can set red-yellow-green color coding based on thresholds. That makes it especially easy to monitor behavior at-a-glance when you’re jogging by the wallboard on the way to your next meeting.

## Works flawlessly with groupings

One of the most powerful features of Sysdig Cloud are our groupings. We can group data via applications, services, data centers, or any other logical set of containers or hosts. Number panels work great with groupings so that you can easily monitor higher level questions like “How is X service performing right now?” 

<a href="/wp-content/uploads/2016/02/Single-Number-Panel-1.png" rel="attachment wp-att-2419"><img src="/wp-content/uploads/2016/02/Single-Number-Panel-1.png" alt="Container Monitoring with Sysdig Number Panel" width="750" height="536" class="alignnone size-medium wp-image-2419" /></a>  
  
Try the Number Panel We’ve already made it GA - so go ahead! 

If you’re not yet a Sysdig Cloud user, log in now for a [free trial][3], and change the way your team looks at data for the better!

 [1]: /wp-content/uploads/2016/02/Number-Panel-Blog-1.png
 [2]: /wp-content/uploads/2016/02/number_panel_feature.gif
 [3]: https://www.sysdigrp2rs.wpengine.com