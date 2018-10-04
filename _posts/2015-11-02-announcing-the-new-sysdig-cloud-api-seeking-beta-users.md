---
ID: 2196
post_title: 'Announcing the New Sysdig Cloud API: Seeking Beta Users'
author: Chris Crane
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/announcing-the-new-sysdig-cloud-api-seeking-beta-users/
published: true
post_date: 2015-11-02 09:00:24
---
For a long time, one of our most requested features has been an API to access various Sysdig Cloud functionality. Well, today I’m excited to announce that the API is here! At this time we’ve completed the core framework of authentication and documentation that should enable us to expand the functionality of the API rapidly going forward. We’ve also released some basic alert management functionality, and we’re seeking beta users who are interested in testing more advanced capabilities. Please contact us as <a href="mailto:support@sysdig.com" target="_blank">support@sysdig.com</a> if you’re interested in participating in the beta. 

Let’s talk about what the API can do already though! First off, you can find full documentation for the API <a href="http://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/205233166-The-Sysdig-Cloud-API-Specification" target="_blank">on our support page</a>. We’ve also published a complete (and interactive!) <a href="https://app.sysdigcloud.com/apidocs/" target="_blank">list of available API calls</a>. 

With these current API calls, you can now programmatically: 
*   Create, edit, and delete alerts
*   Enable and disable alerts

This functionality is expected to support two main use cases, but you may be able to come up with many more! 

*   First off, you can now automatically disable alerts during activity that is expected to cause disruption to your system: for example, code deploys. You can easily build into your code deployment process an automated switch to disable all alerts before the deploy, and enable them some amount of time afterwards. No more barrage of noisey alerts when you’re pushing a big update and are already tuned into your system!
*   And second, you can now build the alert creation process itself into your deployment cycle. It should be easy to create recipes to automatically create and apply the relevant alerts to new nodes, and you can even configure settings and tune the thresholds accordingly.

This is really just the tip of the iceberg though - expect much more to come here in the future! And again, if you’d like to be involved, or if you have any feedback or requests for the Sysdig Cloud API: please don’t hesitate to reach out!