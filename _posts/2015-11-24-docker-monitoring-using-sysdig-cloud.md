---
ID: 2266
post_title: >
  Getting Started with Docker Monitoring
  Using Sysdig Cloud
author: Ranvijay Jamwal
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/docker-monitoring-using-sysdig-cloud/
published: true
post_date: 2015-11-24 09:00:50
---
*
#### Editor’s note: This post from <a href="https://twitter.com/ranvijayj" target="_blank">Ranvijay Jamwal</a>, DevOps Engineer at <a href="http://www.tothenew.com/" target="_blank">TO THE NEW Digital</a>, originally appeared on the <a href="http://www.tothenew.com/blog/devops-docker-monitoring-sysdig-cloud/" target="_blank">TO THE NEW blog</a>.

#### This is part of our series of guest blog posts, written by real Sysdig users telling their own stories in their own words. If you’re interested in posting here, please <a href="mailto:info@sysdig.com?subject=Guest%20post" target="_blank">reach out</a> – we’d love to hear from you!* 

* * *

Sysdig has been one of the most advanced cloud-based tools for monitoring your infrastructure. So, talking about Docker monitoring, Sysdig gives us a lot of insights about our containers, a few of which we will be talking about in this blog. Docker is the most widely used and trending tool when it comes to <a href="http://www.tothenew.com/devops-automation-consulting" target="_blank">DevOps</a>. In this post, I'll show you a step-by-step guide to how I was able to deploy the Sysdig Cloud container within a few minutes in my environment. From there we'll see in a few clicks how you can monitor and alert on individual Docker containers or even clusters of containers. 

## Use Case

In this use-case we had to monitor Docker containers for an <a href="http://www.tothenew.com/case-studies/security-online-fashion-lifestyle-brand" target="_blank">e-commerce web application</a>. We had to find the tool which gives us all insights of the Docker containers. Sysdig was the next tool in <a href="http://www.tothenew.com/devops-chef-puppet-docker" target="_blank">our list of tools and technologies</a> that we are using at TO THE NEW Digital. I have a container running on my host machine and apache2 service running inside it. Let’s monitor this container. 

## Installation Steps

Create a Sysdig account from this URL: <a href="https://sysdigrp2rs.wpengine.com/docker-monitoring/" target="_blank">https://sysdigrp2rs.wpengine.com/docker-monitoring/</a>. You will get a confirmation mail from Sysdig. Click on the activate button, this will redirect you to the Sysdig page where you can further complete your registration. After that click on next, which will take you to the below page: 

<a href="/wp-content/uploads/2015/11/Installation-method.png" data-rel="lightbox-1"> <img src="/wp-content/uploads/2015/11/Installation-method.png" /></a> 

Select Docker Container here which will open a page where installation steps will be mentioned: 

<a href="/wp-content/uploads/2015/11/set-up-environment.png" data-rel="lightbox-2"> <img src="/wp-content/uploads/2015/11/set-up-environment.png" /></a> 

Above commands can be run on your host machine. It is account dependent so the ACCESS_KEY will be different. You can replace [TAGS] option as specified in the above image. I have run the second command on my host machine using -itd switch. Else the container will run interactively and will print all lines on the terminal. Also, I have not used the TAGS switch. The command will now look like: 

<pre>docker run -itd --name sysdig-agent --privileged --net host --pid host -e ACCESS_KEY=8dd53112-d004-4bdc-90a8-bde9f26825a3 -v /var/run/docker.sock:/host/var/run/docker.sock -v /dev:/host/dev -v /proc:/host/proc:ro -v /boot:/host/boot:ro -v /lib/modules:/host/lib/modules:ro -v /usr:/host/usr:ro sysdig/agent
</pre>

After this it should say: 

<a href="/wp-content/uploads/2015/11/agent-connected.png" data-rel="lightbox-3"> <img src="/wp-content/uploads/2015/11/agent-connected.png" /></a> 

Just click on Next and it should take you to the Integrate with AWS page: 

<a href="/wp-content/uploads/2015/11/integrate-AWS.png" data-rel="lightbox-4"> <img src="/wp-content/uploads/2015/11/integrate-AWS.png" /></a> 

If you want any <a href="http://www.tothenew.com/devops-aws" target="_blank">integration with AWS account</a> go ahead and give read access to Sysdig. You can create an IAM user, attach a READ ONLY policy to the user and enter the credentials here. Since, I am just concerned about the Docker monitoring, so will skip this step clicking on SKIP. 

After this it should say: 

<a href="/wp-content/uploads/2015/11/congrats.png" data-rel="lightbox-5"> <img src="/wp-content/uploads/2015/11/congrats.png" /></a> 

## Monitoring

Let’s Head onto the Console to see what we have got. Select Explore and it will show you the following options as in the image below: 

<a href="/wp-content/uploads/2015/11/explore1.png" data-rel="lightbox-6"> <img src="/wp-content/uploads/2015/11/explore1.png" /></a> 

It will show you the host machine with a “+” sign before it. If you select the host machine, it will give you insights of the host machine. I have selected System -> Top Processes and it is showing me all the top processes running on my host machine. You can select any other option from the left hand-side. An important phase of any <a href="http://www.tothenew.com/devops-automation-consulting" target="_blank">Devops project is Docker monitoring</a>. For that just click on the “+” sign adjacent to the host machine’s hostname. It should show something like below: 

<a href="/wp-content/uploads/2015/11/explore2.png" data-rel="lightbox-7"> <img src="/wp-content/uploads/2015/11/explore2.png" /></a> 

So, it has listed both the Docker containers as in the above image. One of them is the sysdig-agent container which is giving us all the insights. Now, we shall select the apachecon container and see what insights we have. Also, in the bottom left I have selected Container Overview and it will give you an overall view of metrics related to the container: 

<a href="/wp-content/uploads/2015/11/explore3.png" data-rel="lightbox-8"> <img src="/wp-content/uploads/2015/11/explore3.png" /></a> 

You can select Top Processes to see the top processes running inside your Docker container. Also, you can then search by process name if there are a lot of processes running. 

<a href="/wp-content/uploads/2015/11/explore4.png" data-rel="lightbox-9"> <img src="/wp-content/uploads/2015/11/explore4.png" /></a> 

## Alerting

Let’s see what type of alerts can we put. 

What we are looking for is monitoring of Docker related things. Select on alert icon there “Add Alert”: 

<a href="/wp-content/uploads/2015/11/cpu-memory.png" data-rel="lightbox-10"> <img src="/wp-content/uploads/2015/11/cpu-memory.png" /></a> 

After clicking on the above it should open something like: 

<a href="/wp-content/uploads/2015/11/new-alert.png" data-rel="lightbox-11"> <img src="/wp-content/uploads/2015/11/new-alert.png" /></a> 

It gives you 4 main topics inside which you can choose some options. Two of them are in the above image. Choosing the metrics depends on the use-case. You can choose metrics like cpu, memory etc. We would be interested in the Docker part of the alerting. One of the options is **Entity is Down**, which can be applied to containers too, and another such metric that Sysdig gives us is **Container Count**. 

<pre>container.count
Count of the number of containers.
</pre> Now, I have selected the condition if Average count < = 2 i.e. average count of containers on my host machine don’t fall in this condition it should alert me. I have selected 2 here because I have only 2 containers running on my host machine. I want to be alerted if the containers count reduces. The next two options Topics are as below and are easy to understand: </p> 

<a href="/wp-content/uploads/2015/11/alert3.png" data-rel="lightbox-13"> <img src="/wp-content/uploads/2015/11/alert3.png" /></a> 

After you are done just select on CREATE as shown in an earlier image. 

** Now, you can put on individual containers. **

Select the container this time and click on the Add Alert button as we did in the previous step. It will again open something like this: 

<a href="/wp-content/uploads/2015/11/alert4.png" data-rel="lightbox-13"> <img src="/wp-content/uploads/2015/11/alert4.png" /></a> 

It will already have the container ID in the scope. Now, I can monitor if my Docker container is down. I can go go ahead and choose Entity is Down and set an alert for that. If containers within the scope are down I will be alerted. Also, I can now get all insights of my Docker containers. 

I personally found Sysdig a good way to monitor my Docker containers. You can see how I was able to deploy the Sysdig Cloud container into my environment in just a couple minutes, and I could automatically see all the other containers I was running. I did not need to do any configuration or manipulate my own containers at all. From there I was able to set up alerts on my containers in just a few clicks. Pretty cool!