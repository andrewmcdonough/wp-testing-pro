---
ID: 4561
post_title: 'Sysdig + IBM Bluemix container service: A rhapsody in BLUEmix…'
author: Dan Papandrea
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-and-ibm-bluemix-container-service-a-rhapsody-in-bluemix/
published: true
post_date: 2017-09-06 13:10:33
---
We at Sysdig were happy to see IBM’s launch of their new container service. A very stable and well managed cluster environment for your apps and containers powered by IBM is a big step forward for both Kubernetes and the Cloud. And, in our humble opinion, pairing Bluemix up with a cornerstone container intelligence tool like Sysdig is a match made in Devops heaven.

This blog was easy to write as it took 3 steps to get up and running with Bluemix’s excellent deployment and management cli tools along with the Sysdig Monitor daemonset deployment.

1.  Deploy your Cluster. <a href="https://console.bluemix.net/docs/containers/cs_tutorials.html#cs_tutorials" target="_blank">Great tutorial here</a>.
2.  Deploy Sysdig: <a href="https://sysdigrp2rs.wpengine.com/sign-up/" target="_blank">Sign up for a free trial</a> on our website, then deploy our daemonset.
3.  Prosper!

For context, Sysdig is the <a href="https://sysdigrp2rs.wpengine.com/" target="_blank">container monitoring company</a>. We’re based on <a href="https://www.sysdig.org/" target="_blank">the open source Linux troubleshooting project by the same name</a>. The open source project allows you to see every single system call down to process, arguments, payload, and connection on a single host. The commercial offering turns all this data into thousands of metrics for every container and host, aggregates it all, and gives you dashboarding, alerting, and an htop-like exploration environment.

Before we dive into the how of using Bluemix and Sysdig, let’s talk about the particular challenges of monitoring containers. [tweet_box design="default" float="none"]IBM #Bluemix and #Sysdig, a match made in DevOps heaven. Get full container visibility in 3 straightforward steps.[/tweet_box] 

## Containers radically change monitoring

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image1.png" alt="Container advantages" width="25%" height="25%" style="float: left; margin: 1em 2em 1em 1em;" /> 
Containers are pretty powerful. They are :

*   Simple - typically an individual process
*   Small - 1/10th of the size of a VM means they’re portable
*   Isolated - They have few dependencies
*   Dynamic - Rapid startup times means they can be scaled, killed, and moved quickly

Keeping containers simple is core to their value proposition and what makes them a great building block for microservices. But this simplicity comes at a cost. From an ops perspective, you need deep visibility inside containers rather than just knowing that some containers exist.

Your containers are also likey managed by an orchestration system (think Bluemix backed by Kubernetes), and your developers may be pushing new applications at any time, without informing the ops team.

Alright, so now we know we’re dealing with these small black boxes that appear, die, and are moved at the whims of your orchestration system. Your developers have the freedom to add and modify their applications freely. And your job is to make sure that your company’s apps are running properly, not to mention have the data to solve issues when they might arise.

Now let’s solve these monitoring challenges using Bluemix and Sysdig!

### Prerequisites

To gain access to your cluster, download and install a few CLI tools and the IBM Bluemix Container Service plug-in.

<a href="https://clis.ng.bluemix.net/ui/home.html" target="_blank">Bluemix CLI</a>

<a href="https://kubernetes.io/docs/tasks/tools/install-kubectl/" target="_blank">Kubernetes CLI</a>

1.  Deploy your IBM Bluemix Container Service via the excellent UI Bluemix provides

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image2.png" alt="IBM Bluemix container" width="624" height="189" class="alignleft size-full wp-image-4575" />][1] <ol start="2">
  <li>
    We install the Bluemix cli plugin. In this example i am using the windows CLI. <a href="https://clis.ng.bluemix.net/ui/home.html" target="_blank">Install instructions here</a>.
  </li>
</ol>

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image3.png" alt="Bluemix windows CLI" width="624" height="214" class="alignleft size-full wp-image-4574" />][2] 


<ol start="3">
  <li>
    We access our cluster, here are directions on how to connect:
  </li>
</ol>

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image4.png" alt="Connect to cluster" width="424" height="347" class="alignleft size-full wp-image-4573" />][3] 


After about 5-10 minutes we have a fully functional Bluemix Kubernetes Cluster:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image5.png" alt="Bluemix Kubernets cluster" width="624" height="241" class="alignleft size-full wp-image-4572" />][4] 


#### Sysdig Installation with DaemonSet (Look Ma no hands!)

Before deploying Sysdig, it’s worth a moment to describe our instrumentation approach because it’s so different from other container monitoring approaches out there. We have developed a transparent, per-host instrumentation model that doesn’t require developers to modify code, doesn’t involve adding a container per pod, and certainly doesn’t involve adding a process to every single container!

This transparent instrumentation - we call it ContainerVision - captures all application, container, statsd, and host metrics with a single instrumentation point, leveraging the tracepoints facility in the kernel, and sends it to a container per host for processing & transfer. This eliminates the need to turn everything into a statsd metric, which is something we’ve seen many people resort to. Unlike per-pod sidecar models, per-host agents drastically reduce resource consumption of monitoring agents, and require no modification to application code. It does, however, require a privileged container and a kernel module.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image6.jpg" alt="Sysdig ContainerVision" width="892" height="600" class="alignleft size-full wp-image-4571" />][5] 


Ok, back to Bluemix. Schedule the Sysdig agent container as a DaemonSet by creating a resource manifest (YAML) file following <a href="https://github.com/draios/sysdig-cloud-scripts/tree/master/agent_deploy/kubernetes" target="_blank">this example provided on github</a>. Any parameters commented out with ‘#’ are optional or are needed only in on-premises installations (We have cloud and on-premise software <a href="https://sysdigrp2rs.wpengine.com/pricing/" target="_blank">offerings</a>.). You must at least enter your Sysdig Monitor agent access key as found in the User Profile tab of the Sysdig Monitor app settings (Go to `Sysdig -> Settings -> User Profile -> Access Key.`):

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image7.png" alt="Bluemix CLI plugin" width="2255" height="381" class="alignleft size-full wp-image-4570" />][6] 


Deploy the DaemonSet by issuing this command:

    kubectl create -f ‘sysdig.yaml’

#### Now time to Sysdig!

Before we hop into the metrics - just a note - **everything you see below is out of the box**. No special configurations, no modification to your application code, no configuring plug-ins. While Sysdig can give you even more than what’s below, you’ll see you start with quite a lot!

We now have an overview of my agent host(s) running my Bluemix Cluster. In less than 20 minutes I have full visibility of my cluster!

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image8.png" alt="Sysdig cluster visibility" width="624" height="195" class="alignleft size-full wp-image-4569" />][7] 


As you can see I am seeing CPU, Memory, and Network details by selecting a node. Then, drilling using Sysdig’s Out of the box **“Overview by Process”**, I get down to a default dashboard showing per-process metrics.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image9.png" alt="Sysdig overview by process" width="2500" height="1393" class="alignleft size-full wp-image-4568" />][8] 


I deployed a simple <a href="https://github.com/kubernetes/kubernetes/tree/master/examples/guestbook-go" target="_blank">Guestbook-GO from the Kubernetes example pages</a> to the DEFAULT namespace and now I want to view Kubernetes as a whole vs at a selective host by host basis. We have the bases covered in terms of Out of the box visualization of Deployments, Namespaces, Pods, and Services:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image10.png" alt="Deployments, Namespaces, Pods, and Services" width="215" height="207" class="alignleft size-full wp-image-4567" />][9] 


#### Here’s a Kubernetes overview of my cluster

Notice Sysdig is grabbing the metatag data directly from Kubernetes API without any type of direct configuration. I’m seeing our request count per service, Top Services, Top namespaces and host capacity. This helps me grow the environment naturally and improved capacity planning based on **REAL** metrics.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image11.png" alt="Sysdig Kubernetes metadata" width="2499" height="1305" class="alignleft size-full wp-image-4566" />][10] 


I can even view how my applications are communicating through our great topology visualizations, here’s a network topology view:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image12.png" alt="Sysdig topology view" width="613" height="444" class="alignleft size-full wp-image-4565" />][11] 


#### What about application metrics? How do I get to those?

Here’s the same redis example from the guestbook, we have a service level view, not just single hosts, <a href="https://sysdigrp2rs.wpengine.com/product/integrations/" target="_blank">and many many more</a>.

Similarly, Sysdig can visualize statsd, JMX, and prometheus custom metrics… without even changing the collection endpoint you already have in place. (Yes, that sounds crazy. Read how it works <a href="https://sysdigrp2rs.wpengine.com/blog/how-to-collect-statsd-metrics-in-containers/" target="_blank">here</a>.)

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image13.png" alt="Sysdig statsd JXM prometheus" width="1452" height="942" class="alignleft size-full wp-image-4564" />][12] 


#### All of that in 3 steps!

IBM Container Service is a great place for developers hosting applications and running production applications. Sysdig makes it even better with true visibility and intelligence. We even have a <a href="https://sysdigrp2rs.wpengine.com/blog/monitoring-kubernetes-with-sysdig-cloud/" target="_blank">4 part tutorial on monitoring kubernetes with Sysdig</a> here.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image14.png" alt="Sysdig and IBM bluemix" width="446" height="216" class="alignleft size-full wp-image-4563" />][13]

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image2.png
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image3.png
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image4.png
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image5.png
 [5]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image6.jpg
 [6]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image7.png
 [7]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image8.png
 [8]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image9.png
 [9]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image10.png
 [10]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image11.png
 [11]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image12.png
 [12]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image13.png
 [13]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/09/image14.png