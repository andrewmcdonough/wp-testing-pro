---
ID: 4604
post_title: 'Sysdig Inspect &#8211; a powerful interface for container troubleshooting and security investigation'
author: Loris Degioanni
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-inspect/
published: true
post_date: 2017-09-23 10:51:57
---
Sysdig is routinely praised by users for the richness of the data it’s able to capture and for the ability to store system, application and container that data into capture files that can be easily shared with other people.

However, extracting insights from rich data can be hard. Mastering the art of analyzing sysdig capture files requires dedication and skills. This is why we constantly try to improve workflows around sysdig and find ways to get better insights with less effort. Today, we bring these efforts to a new level with the release of Sysdig Inspect.

Sysdig Inspect is a powerful, intuitive tool for sysdig capture analysis that runs natively on your Mac or your Linux PC, with a user interface that has been designed for performance and security investigation.

Sysdig Inspect, and the experience of using it, are really best explained by trying it, which requires just a [30 second installation][1]. But in case that’s too much for you, here’s a one minute video to get you started:

<div style="text-align:center, clear:both">
  <iframe width="560" height="315" src="https://www.youtube.com/embed/M1W8txpJKxY" frameborder="0" allowfullscreen></iframe>
</div>

Let me share a few of the principles that we used to guide the design of Sysdig Inspect.

### Instant highlights

[<img style="max-width: 100%; height: auto;" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/Sysdig-Inspect-1.png" alt="Sysdig Inspect Instant Highlights" width="1200" height="750" class="alignleft wp-image-4350" />][2] 

The overview page offers an out of the box, at a glance summary of the content of the capture file. Content is organized in tiles, each of which shows the value of a relevant metric and its trend. Tiles are organized in categories to surface useful information more clearly and are starting point for investigation and drill down. 
### Sub-second microtrends and metric correlation

[<img style="max-width: 100%; height: auto;" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/Sysdig-Inspect-2.png" alt="Sysdig Inspect Sub-second microtrends and metric correlation" width="1200" height="750" class="alignleft wp-image-4350" />][3] 

Once you click on a tile, you will see the sub-second trend of the metric shown by the tile. Yes, sub-second. You will be amazed at how different your system, containers and applications look at this level of granularity. Multiple tiles can be selected to see how metrics correlate to each other and identify hot spots. 
### Intuitive drill-down-oriented workflow

[<img style="max-width: 100%; height: auto;" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/Sysdig-Inspect-3.png" alt="Sysdig Inspect Intuitive drill-down-oriented workflow" width="1200" height="750" class="alignleft wp-image-4350" />][4] 

You can drill down into any tile to see the data behind it and start investigating. At this point you can either use the timeline to restrict what data you are seeing, or further drill down by double clicking on any line of data. You will be able to see processes, files, network connections and much more. 
### All the details when you need them

[<img style="max-width: 100%; height: auto;" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/Sysdig-Inspect-4.png" alt="Sysdig Inspect All the details when you need them" width="1200" height="750" class="alignleft wp-image-4350" />][5] 

Every single byte of data that is read or written to a file, to a network connection to a pipe is recorded in the capture file and Sysdig Inspect makes it easy to observe it. Do you need to troubleshoot an intermittent network issue or determine what a malware wrote to the file system? All the data you need is there. And, of course, you can switch at any time into sysdig mode and look at every single system call.

### Conclusion Most of all, the guiding principle when designing Sysdig Inspect was: make troubleshooting and security investigation easy, effective and, as much as possible, a pleasure. Either if you took the capture file manually or you used 

[Sysdig Monitor][6] to launch a capture just when an alert triggered, we aim at providing all the tools you need to monitor, troubleshoot and do forensics in your container platform. Did we succeed achieving this goal? You can judge by yourself by trying it! To make your life easy, here [are some capture files that you can use to play with it][7]. 
### What's next?

*   Tune into the [Sysdig Camp-Con-World-Fest-Summit Livestream][8] for two days of new announcements and hands on container troubleshooting sessions
*   [Download Sysdig Inspect][1]
*   Join our [Slack group][9] and let us know what you think!

 [1]: https://github.com/draios/sysdig-inspect/
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/Sysdig-Inspect-1.png
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/Sysdig-Inspect-2.png
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/Sysdig-Inspect-3.png
 [5]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/Sysdig-Inspect-4.png
 [6]: https://sysdigrp2rs.wpengine.com/blog/troubleshooting-containers-when-theyre-gone/
 [7]: https://github.com/draios/sysdig-inspect/tree/master/capture-samples
 [8]: https://go.sysdigrp2rs.wpengine.com/ccwfs2017
 [9]: http://slack.sysdigrp2rs.wpengine.com