---
ID: 3620
post_title: New opensource Sysdig cheatsheet
author: Knox Anderson
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/opensource-sysdig-cheatsheet/
published: true
post_date: 2017-01-20 09:59:53
---
We've introduced an updated [opensource sysdig cheatsheet][1] of our favorite commands for getting deep visibility into containers and linux servers!

Sysdig started off with a simple goal in mind: “**To give you easy access to the actual behavior of your Linux systems**”.

One prime example is the missing link between the network and the host. There are tools that let you see what’s happening on the wire, and different tools that let you look at activity inside your machines. But even though the two worlds are deeply related, the picture you get today is disconnected at best. So disconnected, in fact, that the simple task of understanding **which process** is sending data to the network is something that has eluded a clean solution.

With sysdig you can do this:

<script src="https://gist.github.com/KnoxAnderson/8677014364d38d7ec8e271cf0709795e.js"></script> 
Or, if you like a simpler view, this:

<script src="https://gist.github.com/KnoxAnderson/b70a2e68ca3a62da1c95d8809a05a8d1.js"></script> 
Around the same time as the creation of sysdig a company named DotCloud developed [Docker][2], and [the rest is history][3]. If you are new to sysdig or troubleshooting containers this [linux troubleshooting cheatsheet][4] is a great transition from tools like htop, strace, tcpdump, and other tools you've likely used in the past.

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/01/Sysdig_cheat_sheet_2017_download_version-2.pdf
 [2]: https://www.docker.com/
 [3]: https://sysdigrp2rs.wpengine.com/blog/let-light-sysdig-adds-container-visibility/
 [4]: https://go.sysdigrp2rs.wpengine.com/troubleshooting-linux-cheatsheet