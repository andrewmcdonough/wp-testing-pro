---
ID: 6088
post_title: >
  Three Ways Red Hat Acquiring CoreOS
  Helps Cloud Native
author: Michael Ducy
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/three-ways-red-hat-acquiring-coreos-helps-cloud-native/
published: true
post_date: 2018-01-30 15:10:27
---
﻿It was <a href="https://www.redhat.com/en/about/press-releases/red-hat-acquire-coreos-expanding-its-kubernetes-and-containers-leadership" target="_blank">announced today</a> that Red Hat has reached an agreement to purchase <a href="https://coreos.com" target="_blank">CoreOS</a>. In many ways this acquisition makes a lot of sense. CoreOS has contributed greatly to open source communities since its founding, helping to <a href="https://github.com/coreos/" target="_blank">establish projects</a> such as etcd, Prometheus, and rkt (as well as more). While it’s still to early to tell what this exactly means for CoreOS, we at Sysdig see three ways this is a net positive for the Cloud Native community. 

## Container Linux becomes the Enterprise Linux Container Distribution {#containerlinuxbecomestheenterpriselinuxcontainerdistribution}

With the advent of containers, one question that stands out for many organizations is “What about my host OS?” <a href="https://coreos.com/os/docs/latest/" target="_blank">Container Linux</a> from CoreOS sought to revolutionize the idea of the container host operating system by minimizing what was available in the base operating system, and by treating every application ran on the host OS as a container. For a company like Red Hat, whose revenue has long relied on enterprise subscriptions for its Enterprise Linux, threats to the container host OS are very real. As users no longer needed all of the features and functionality a full blown enterprise distribution, they often sought other options, and thus presented the possibility of impacting Red Hat’s revenue stream. [tweet_box design="default" float="none"]This is how #RedHat acquiring #CoreOS helps #CloudNative[/tweet_box] Additionally, companies like Docker have started their own projects to replace the container host OS, namely <a href="https://github.com/linuxkit" target="_blank">LinuxKit</a>. The purchase of CoreOS gives Red Hat a solid entrant in the container host OS space. They can begin to wrap Red Hat’s years of managing an enterprise grade distribution around Container Linux, providing customers with the stability and reliability they expect from a Red Hat distribution. 

## Enterprise Kubernetes Distributions Become Real {#enterprisekubernetesdistributionsbecomereal}

With the acquisition of CoreOS, Red Hat picks up Tectonic, CoreOS’ enterprise Kubernetes distribution. Of course Red Hat already has an enterprise Kubernetes distribution in OpenShift. For end users that have already made OpenShift their distro of choice, this will validate their decision. They should also expect to see the features that made Tectonic unique start to get rolled into their OpenShift platforms, such as integration with CoreOS’ <a href="https://coreos.com/quay-enterprise/" target="_blank">Quay enterprise container registry</a>, chargeback capabilities, and Tectonic’s Open Cloud Services. Additionally, Red Hat has a history of open sourcing proprietary software it acquires. Hopefully this trend will continue with the software acquired from CoreOS. 

## Red Hat Fortifies their Container Runtime Strategy {#redhatfortifiestheircontainerruntimestrategy}

Red Hat has has been contributing heavily to the <a href="https://github.com/opencontainers" target="_blank">Open Container Initiative</a> (OCI) as well as other container runtime projects such as <a href="http://cri-o.io/" target="_blank">CRI-O</a>. They’ve made their intentions very clear in that they wish to offer open alternatives to the defacto standard Docker runtime (which opened it's runtime under <a href="https://github.com/moby" target="_blank">Moby</a>). The acquisition of CoreOS compliments Red Hat’s work in this area, strengthening their portfolio of projects with software such as rkt, as well as the engineering acumen to build a new standard container runtime. From an end user’s perspective choice can often be good, and this part of the acquisition gives the end user more choice in the long run when it comes to container runtimes. 

Overall, the acquisition of CoreOS feels like a win-win for the Cloud Native community. It validates that this is the next area of the industry to invest in, which will help influence enterprises worldwide to accelerate their Cloud Native strategies. Red Hat will make significant investments in improving not only the container host OS experience, and the container runtime experience which in the end will be beneficial for everyone in the community. We for one congratulate the entire CoreOS for what they've contributed to the community over the years, and we are excited for what they will do next.