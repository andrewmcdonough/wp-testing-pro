---
ID: 1598
post_title: 'Announcing csysdig &#8212; think strace + htop + Lua + container support'
author: Loris Degioanni
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/announcing-csysdig-strace-htop-lua-container-support/
published: true
post_date: 2015-06-04 05:45:34
---
A little over one year ago, <a href="https://sysdigrp2rs.wpengine.com/announcing-sysdig/" target="_blank">we announced the first release of sysdig</a>. Since then we have been overjoyed with the great response from developers, administrators and the community at large. During the last 12 months we also received a lot of feedback, much of which has had to do with making the rich metrics collected by sysdig even more accessible and immediate. We took that feedback to heart. Today it is my great pleasure to announce a major new open source sysdig component: csysdig. 

In a single phrase: csysdig offers a simple, intuitive, and fully customizable curses UI for sysdig. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/06/csysdig-screenshot-1024x571.png" alt="csysdig screenshot" width="1024" height="571" class="alignnone size-large wp-image-1608" />][1] 

Csysdig, and the experience of using it, are really best explained by trying it, which requires just a <a href="https://github.com/draios/sysdig/wiki/How%20to%20Install%20Sysdig%20for%20Linux" target="_blank">30 second installation</a>. But if you’re not quite ready to take the plunge, here’s a 3 minute video to get you started: 

<div style="text-align:center">
  http://youtu.be/UJ4wVrbP-Q8
</div>

As the video says, you can imagine csysdig as a combination of many of the command line monitoring tools you already love: tools like strace, tcpdump, htop, iftop and lsof. But, while taking inspiration from many of these tools, we also wanted csysdig to build on the unique advantages and characteristics of sysdig’s technology. Let me share a few of the principles that we used to guide the design of csysdig. 

## One tool to rule them all 

Csysdig offers visibility into a broad range of metrics, including CPU, memory, disk I/O, network I/O, application activity and much more. Our (admittedly very ambitious) goal is to create a single tool, with a consistent interface, that can replace most of the tools you need in your everyday monitoring and troubleshooting. 

## Support for both live analysis and trace file analysis 

The full csysdig goodness is available for live data, or when you open a sysdig trace file, whether captured on the same machine or on another machine entirely. This makes it possible to decouple investigations from being physically logged into the machine where a problem is occurring. You can even analyze trace files from your MAC! 

## An intuitive drill-down-oriented user interface 

The csysdig UI is designed with exploration and troubleshooting in mind. You will be able to quickly move from a system overview to the finest details of a thread’s activity, with very few keyboard clicks. 

For example, with three clicks you can go from seeing the list of running processes, to visualizing the network connections of a specific process, to observing the data exchanged on one of the visualized connections. 

## Rich container support, by design 

Seriously, if you use containers for anything, then we really think you’ll love this tool. We are big fans of containers, and we strongly believe that good visibility is critical to making containers viable and mainstream. This is our contribution. Csysdig offers an overview of container activity, but it also makes it possible to drill down into a single container and observe CPU, disk I/O, network and application activity. With the highest granularity, and all without instrumenting or configuring a single thing in your containers. I promise you’ve never seen anything like this. 

## Fully customizable architecture 

Can’t find the view or the metric you need? Would you prefer a different sorting order in the container view? Do you want to see connections instead of threads when you drill down into a process? No problem: <a href="https://github.com/draios/sysdig/wiki/csysdig-View-Format-Reference" target="_blank">you can make your own view, or modify one of the existing ones</a>, using Lua. 

## Conclusion 

I’d like to thank our good friend, <a href="https://github.com/kristopolous" target="_blank">Chris</a>, who contributed in a major way to the development of csysdig. Chris, myself, and the rest of the Sysdig team have been working really hard on this, putting many evenings and weekends on it. We are very excited to bring it to you, and to hear what you think. 

Here at Sysdig, we spend a big part of our days in front of command line monitoring tools; when building csysdig, we tried to create a tool that we would love to use. We believe interfaces should be designed to make people productive. They should be designed with real use cases in mind. Even if they are just command line interfaces. I hope these principles show through, and I hope csysdig can make your day a little better.

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/06/csysdig-screenshot.png