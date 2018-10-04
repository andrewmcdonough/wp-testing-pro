---
ID: 2788
post_title: Sysdig is now in the DC/OS Universe
author: Apurva Dave
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-now-dcos-universe/
published: true
post_date: 2016-07-07 17:35:24
---
Many of our customers and community members deploy and monitor Docker containers using DC/OS, and more so since DC/OS went open source.

We support pretty sophisticated monitoring of DC/OS, as well as the underlying Mesos and Marathon components, whether you’re monitoring docker containers or another container format. In order to make it even easier, we’ve now added Sysdig to the DC/OS Universe.

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/07/DCOS_Universe_Sysdig.png"></a>[<img alt="Slack Overlay" class="alignnone" height="536" src="/wp-content/uploads/2016/07/DCOS_Universe_Sysdig.png" width="750" />][1] 

This integration allows you to deploy the sysdig-agent container to each slave node by simply providing your Sysdig Cloud API key and other optional information within the DC/OS UI. (FYI, it’s normally the information you’d include in the sysdig-agent docker run command.)

Note you still need to deploy a Sysdig agent to the DC/OS or Mesos Master node. We haven’t automated that yet, but we will! But, not to fear, that’s also a simple process. In fact for standard installations it’s a single line command. [Check out our docs on the matter][2] and you’ll see for yourself.

With this integration, it’s easier than ever to see your entire universe with Sysdig!

 [1]: /wp-content/uploads/2016/07/DCOS_Universe_Sysdig.png
 [2]: http://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/207886103-Sysdig-Install-Mesos-Marathon-DCOS