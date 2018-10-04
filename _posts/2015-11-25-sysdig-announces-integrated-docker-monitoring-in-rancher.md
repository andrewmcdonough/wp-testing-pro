---
ID: 2290
post_title: >
  Sysdig announces integrated Docker
  monitoring in Rancher
author: Chris Crane
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-announces-integrated-docker-monitoring-in-rancher/
published: true
post_date: 2015-11-25 14:00:15
---
Here at Sysdig we build monitoring and visibility tools, specializing in Docker and containerized infrastructures. Our open source CLI tool, <a href="http://www.sysdig.org/" target="_blank">sysdig</a>, offers universal system visibility into Linux machines along with native support for Docker. And based on the same core technology, <a href="https://sysdigrp2rs.wpengine.com/" target="_blank">Sysdig Cloud</a> offers the first and only comprehensive monitoring solution built from the ground up for containers and microservices. In a nutshell, Sysdig’s unique technology let’s you see inside your containers from the outside - no instrumentation, no plugins, no configuration. 

But Docker containers alone are not enough to build a modern microservices-oriented infrastructure: you need proper orchestration and management on top. Enter <a herf="http://rancher.com/" target="_blank">Rancher</a>. Rancher makes it easy to deploy Docker across your architecture and realize all the benefits of a containerized application, without any of the hassle. Not only are we huge fans of Rancher here at Sysdig, but many of our users are using Rancher as well. That’s why, today, we’re excited to announce that Sysdig is integrating our products directly into Rancher! 

With this new integration, which became available with the release of Rancher 0.47, open source sysdig and Sysdig Cloud are now both available in the <a href="http://docs.rancher.com/rancher/rancher-ui/applications/catalog/" target="_blank">Rancher Catalog</a>. The Rancher Catalog is an awesome service provided with the Rancher platform that lets you one-click deploy popular applications directly into your stack. Rancher takes care of all the Docker image management, deployment, scheduling and orchestration for you! So now, with the power of Sysdig + Rancher, in just a couple seconds you can have deep, container-native visibility and monitoring deployed across your entire infrastructure. 

Let’s take a quick look at the deployment process... 

## Deploying sysdig and Sysdig Cloud from the Rancher Catalog

First navigate to the Catalog in your Rancher application and search for “Sysdig”. You should see two options: sysdig and Sysdig Cloud. 

<a href="/wp-content/uploads/2015/11/Rancher_sysdig1.png" data-rel="lightbox-0"> <img src="/wp-content/uploads/2015/11/Rancher_sysdig1.png" /></a> 

Select the application you want to deploy, and all you have to do is add some basic configuration data and you’re good to go! If you’re deploying Sysdig Cloud, you’ll need to add your unique agent key, which can be retrieved from your Sysdig Cloud application. But to deploy the open source sysdig CLI, all you have to do is give your new stack a name: 

<a href="/wp-content/uploads/2015/11/Rancher_sysdig2.png" data-rel="lightbox-1"> <img src="/wp-content/uploads/2015/11/Rancher_sysdig2.png" /></a> 

And that’s it! Now we can see sysdig has been successfully deployed across our entire cluster. Pretty cool right? 

<a href="/wp-content/uploads/2015/11/Rancher_sysdig3.png" data-rel="lightbox-2"> <img src="/wp-content/uploads/2015/11/Rancher_sysdig3.png" /></a> 

You’ll note that Rancher is smart enough to automatically deploy just a single Sysdig container on each host machine in your environment. From there, Sysdig is able to monitor every other container on the machine. We call this patent-pending technology *ContainerVision<sup>TM</sup>* - we’re able to automatically detect every container on a machine, automatically discover the applications running inside each container, and automatically start pulling the relevant application level metrics for your environment. 

We’re really excited about the potential for this Sysdig+Rancher integration, and I hope you’ll go check it out and <a href="https://twitter.com/sysdig" target="_blank">let us know your feedback</a>. If you’re a Rancher user and want to test out Sysdig Cloud, then come sign up for a <a href="https://sysdigrp2rs.wpengine.com/landing-page/" target="_blank">free trial today</a>! And if you end up wanting to make a purchase, let us know you read this blog post and we can offer you 10% off the final price. 

Thanks, happy digging!