---
ID: 5733
post_title: 'System calls never lie: New integrated troubleshooting in Sysdig Monitor'
author: Eric Carter
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/new-integrated-troubleshooting-sysdig-monitor/
published: true
post_date: 2018-01-10 16:15:21
---
<span style="font-weight: 400;">Being able to see the status and performance of your containers in production deployments is critical. But once you see a problem, then what? Most monitoring systems – whether they are open source systems like Prometheus or Grafana, or commercial systems like Datadog – provide no integrated troubleshooting. These solutions leave you to your own devices to cobble together some other tools that actually figure out <i>why</i> a problem occurred.<br /><br /></span><span style="font-weight: 400;">The latest release of <a href="https://sysdigrp2rs.wpengine.com/product/monitor/">Sysdig Monitor</a> delivers integration with <a href="https://github.com/draios/sysdig-inspect/">Sysdig Inspect,</a> our open-source interface for container troubleshooting and security investigation. With this unified approach, you have both visibility into what's happening and why. By combining powerful container-native monitoring and alerting with GUI-based system call analysis, you can now quickly discover and analyze events in your microservices environment. This provides a huge time savings when performing root cause analysis.</span>

## How integrated troubleshooting works {#sysdiginspectoverviewanoverviewofeverythinghappeningonyourbox}

<span style="font-weight: 400;">Sysdig Monitor users can automatically trigger <a href="https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/204190819-Sysdig-Trace-Capture">Sysdig capture files</a> (.scap files) based on any alerting condition. Alerts can be based on system and container performance metrics, custom Prometheus metrics, application metrics, and even events like orchestrator scaling. Capture files contain all of the system calls and OS events that occurred before, during and after a system event. A link automatically added to each Sysdig capture listing lets you instantly open the file in Sysdig Inspect, while preserving the context of your troubleshooting exercise. This new integration means no more logging into production hosts to troubleshoot, and no more copying and moving files to open into a separate tool.<br /><br /> <a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/01/crashloop-capture3-1.gif"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/01/crashloop-capture3-1.gif" alt="integrated troubleshooting and monitoring containers with Sysdig" width="1425" height="747" class="alignnone wp-image-5739 size-full" /></a><br /><br /></span>

<span style="font-weight: 400;">The </span>[<span style="font-weight: 400;">Sysdig Inspect user interface</span>][1]<span style="font-weight: 400;"> is designed to intuitively navigate data-dense Sysdig captures. Its overview page presents an at a glance summary of the system, network, and application activity organized in tiles. Each tile shows the value of a relevant metric and its trend.</span><span style="font-weight: 400;"></span>

<span style="font-weight: 400;">Sysdig Inspect category tiles are your starting point for investigation. They enable you to correlate metrics, zoom in on a specific time slice to isolate conditions, and further drill down to help you get a clear picture of what caused a problem. Customers tell us that having integrated troubleshooting with this level of control in one interface accelerates their time-to-resolution for container issues by 10x or more!<br /></span><span style="font-weight: 400;"><br />[tweet_box design="default" float="none"]How to speed #container #troubleshooting by 10x with Sysdig Monitor and Sysdig Inspect[/tweet_box]<br /></span>

## Troubleshoot CrashLoopBackOff in Kubernetes with Sysdig Monitor and Sysdig Inspect {#sysdiginspectoverviewanoverviewofeverythinghappeningonyourbox}

<span style="font-weight: 400;">Here’s an example of what’s possible now that these tools are integrated. You can, for instance,<a href="https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/204013259-Alerting-and-Notifications"> set an alert</a> to issue a notification and create a capture file whenever a<a href="https://kubernetes.io/docs/concepts/workloads/pods/pod/" target="_blank" rel="noopener"> Kubernetes pod</a> restarts too frequently – usually indicating a <i>CrashLoopBackoff</i> condition. Each time your Kubernetes cluster experiences this state, Sysdig Monitor issues a notification (via UI, email, Slack channel, etc.). It simultaneously triggers a trace capture for the duration of time you've set in your alert. By default, these captures are stored in the Sysdig Monitor AWS S3 bucket. You can optionally choose to store trace files in your own S3 bucket.</span>  
  
[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/01/sysdig-alerts-and-captures-1024x518.png" alt="setting alerts for monitoring and troubleshooting" width="1024" height="518" class="alignnone wp-image-5756 size-large" />][2]  
  
<span style="font-weight: 400;">Once a capture is taken, log in to Sysdig Monitor, select the captures tab and click on the relevant capture file. Sysdig Inspect opens to show summary information from the capture. From here you can explore its content to see what’s causing the problem in your Kubernetes environment. <br /><br /> <a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/01/sysdig-inspect-tiles.png"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/01/sysdig-inspect-tiles-1024x554.png" alt="Sysdig Inspect summary tiles" width="1024" height="554" class="alignnone size-large wp-image-5759" /></a><br /></span><span style="font-weight: 400;"></span>

<span style="font-weight: 400;">In our example, we can quickly see that 12 containers died and were restarted. As a next step, you can drill into these tiles to see what are the affected containers. We see that some containers are having trouble and are dying shortly after being created.</span>

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/01/affected-nginx-containers.png" alt="Affected NGINX containers" width="929" height="299" class="alignnone wp-image-5760 size-full" />][3]

<span style="font-weight: 400;">Why is this happening? One thing to look at first are the processes executing during this time slice. Let's look at NGINX as a start. Here we can simply filter to get a list of the processes executed for the containers – nginx.</span>

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/01/nginx-processes-1024x317.png" alt="NGINX processes" width="1024" height="317" class="alignnone size-large wp-image-5762" />][4]

<span style="font-weight: 400;">From here select one of these nginx processes and troubleshoot the I/O streams to see if there were any errors here. We can see the different DNS requests nginx tries to do before crashing. In this case Sysdig Inspect reveals what happened to that process before dying – </span>*<span style="font-weight: 400;">host not found in /etc/nginx/nginx.conf. </span>*<span style="font-weight: 400;">Looks like we've found the problem! In just few clicks we were able to determine the cause of our pod restart issue even though these particular containers no longer exist!</span>

## <img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/01/nginx-errors.png" alt="integrated troubleshooting of NGINX errors" width="890" height="428" class="alignnone wp-image-5765 size-full" style="font-size: 16px;" />  
  
Give it a try<span style="font-weight: 400;"></span> {#sysdiginspectoverviewanoverviewofeverythinghappeningonyourbox}

<span style="font-weight: 400;">We hope this quick walkthrough gives you some idea of what is now possible with the new integrated troubleshooting. If you’re a Sysdig Monitor customer or currently taking advantage of our free trial, good news, you can try this today. Take a capture and click thru to check out how it works. If you'd like to explore the .scap file we used above, you can <a href="https://go.sysdigrp2rs.wpengine.com/l/231542/2018-01-10/8cjzw/231542/25456/pod_crashloop_2aaa1032_0520_41c9_8851_5a4bd0283e40_1515161669361.scap.zip">download it here</a>. To see more of how Sysdig Inspect works, check out this <a href="https://youtu.be/M1W8txpJKxY" target="_blank" rel="noopener">overview video from our YouTube channel</a>.<br /></span>

 [1]: https://sysdigrp2rs.wpengine.com/blog/sysdig-inspect-explained-visually/
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/01/sysdig-alerts-and-captures.png
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/01/affected-nginx-containers.png
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/01/nginx-processes.png