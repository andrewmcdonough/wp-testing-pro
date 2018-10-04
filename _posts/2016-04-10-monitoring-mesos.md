---
ID: 2521
post_title: >
  Monitoring Mesos, Marathon, and DCOS
  with Sysdig
author: Alex Fabijanic
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/monitoring-mesos/
published: true
post_date: 2016-04-10 23:51:22
---
As we announced as part of our [partnership][1] today, Sysdig and Sysdig Monitor now provide full metrics and metadata support for [Apache Mesos][2], [Mesosphere Marathon framework][3], and the Mesosphere Data Center Operating System. That means you can see deep into the applications running in your Mesos-based environment, and also use framework metadata to shift from monitoring physical resources (like hosts and containers) to monitoring logical resources (like apps and services).

Let’s take a look into these platforms and also apply Sysdig’s integration to do some troubleshooting.

### Mesos

Mesos is a platform for fine-grained resource sharing. It is a distributed systems kernel, built on the same principles as the Linux kernel, but at a higher level of abstraction - it abstracts CPU, memory, storage, and other compute resources away from physical or virtual machines, enabling fault-tolerant and elastic distributed systems to be built easily and run effectively. The Mesos kernel runs on every cluster machine and provides a programming interface for resource management and scheduling across cloud environment. Mesos enables a fine-grained sharing across diverse cluster computing frameworks, by giving frameworks a common interface for accessing cluster resources.

Mesos runs on physical or virtual machines which can act as master (orchestrating entity, one at a time) or slave (worker, many at a time). A node can act as both master and slave, but typically the roles are divided and one node will have one role. The main Mesos entities are frameworks and tasks - Mesos is extended by custom “plugins” called frameworks; tasks are natively isolated (Docker or Mesos containers) units of execution belonging to frameworks (orphaned tasks can happen, but that is beyond the scope of this article).

### Marathon

Marathon is a popular (Mesosphere-sponsored) open source Mesos framework. Marathon classifies, schedules and keeps running services in the cloud, using the resources managed by Mesos. Marathon groups Mesos tasks into apps, which are in turn classified in groups. Groups are hierarchical and recursive, so every group contains one or more apps and groups.

### DC/OS

DCOS (Data Center Operating System) takes Mesos and Marathon, and bundles additional services. Add-on modules like mesos-dns, tooling like a CLI, a GUI, a repository for the packages that you want to run, and frameworks like Marathon (a.k.a. distributed init), Chronos (a.k.a. distributed cron), and more.

### Sysdig monitoring integration

Our open source products sysdig and csysdig, as well as our commercial offering **Sysdig Monitor** now provide full support for **Mesos**, **Marathon**, and **DCOS**. The most common cases work “out-of-the-box”, without any configuration effort.

With our integration, cluster metadata for system layout as well as performance indicators metrics are collected, displayed and saved for later replay of the state and behavior of the system. While metadata provides insight into the cluster layout, configuration and status, metrics show performance indicators of what’s performing properly and what’s not.. Without further ado, let’s look at Sysdig in action:

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/04/mesos-img-1.png"></a>[![sysdig][4]][4] 

In the upper portion of the screenshot in Figure 1, we can see containers classified under a tree structure indicating their location in the cluster hierarchy. Since the filtering is set to include Marathon metadata, frameworks other than Marathon, will show “n/a” in place of Marathon group and app entry.

In the lower portion of the screen, Marathon group aggregate metrics (Container Count, CPU%, Memory% etc.) are shown. As you drill in deeper, Sysdig will give you application-level metrics in addition to container metrics; you can read more about that [here][5].

To make things interesting and demonstrate the troubleshooting capabilities of this new Sysdig feature, let us do something silly with the Mesos/Marathon cluster and then troubleshoot the issue. Let us say that a developer needs a redis server in the cluster and decides to instantiate it in a single Mesos container instance.

At the same time, the sysadmin sets to install the redis server in docker container. To make sure there is enough parallelism and redundancy available, he configures it to run an instance on every cluster node. They both configure host networking to be used - a very simplified and silly scenario, for sure, but you can see where this is going - there will be a conflict somewhere when a redis server in docker container tries to bind to an endpoint already taken by the redis server in mesos container. And it will manifest like this in Sysdig Monitor:

[embed width="600" height="350"]https://sysdigrp2rs.wpengine.com/wp-content/uploads/2016/04/Mesos-blog-1-1.mp4[/embed]

Notice the blinking “n/a” tree item at the bottom of the screencast above - there is something wrong with one of the redis tasks and there are only two redis tasks in docker containers but there are three nodes. The reason for the blinking “redis-in-mesos-container” on the Sysdig Monitor UI puzzles our sysadmin.

Running open source sysdig on the failing task Mesos node quickly reveals the root of problem - there’s someone already listening on port 6379:

<pre># sysdig container.image="redis"
737570 21:46:36.683362095 1 redis-server (14098) > socket domain=10(AF_INET6) type=1 proto=6
737574 21:46:36.683367602 1 redis-server (14098) &lt; socket fd=4(&lt;6>)
737575 21:46:36.683368854 1 redis-server (14098) > setsockopt
737576 21:46:36.683371130 1 redis-server (14098) &lt; setsockopt
737577 21:46:36.683371612 1 redis-server (14098) > setsockopt
737578 21:46:36.683372235 1 redis-server (14098) &lt; setsockopt
737579 21:46:36.683373639 1 redis-server (14098) > bind fd=4(&lt;6>)
<strong>737582 21:46:36.683378450 1 redis-server (14098) &lt; bind res=-98(EADDRINUSE) addr=family 10
</strong>737589 21:46:36.683386855 1 redis-server (14098) > close fd=4(&lt;6>)
737594 21:46:36.683397070 1 redis-server (14098) &lt; close res=0
737596 21:46:36.683398407 1 redis-server (14098) > switch next=31264(docker) pgft_maj=0 pgft_min=5452 vm_size=20808 vm_rss=1776 vm_swap=0
737644 21:46:36.683439419 1 redis-server (14098) > socket domain=2(AF_INET) type=1 proto=6
737645 21:46:36.683443054 1 redis-server (14098) &lt; socket fd=4(&lt;4>)
737646 21:46:36.683443523 1 redis-server (14098) > setsockopt
737648 21:46:36.683444296 1 redis-server (14098) &lt; setsockopt
737649 21:46:36.683444721 1 redis-server (14098) > bind fd=4(&lt;4>)
&lt;/strong><strong>737650 21:46:36.683446296 1 redis-server (14098) &lt; bind res=-98(EADDRINUSE) addr=0.0.0.0:6379
</strong>737653 21:46:36.683448595 1 redis-server (14098) > close fd=4(&lt;4>)
737654 21:46:36.683449177 1 redis-server (14098) &lt; close res=0
737658 21:46:36.683454255 1 redis-server (14098) > gettimeofday
737659 21:46:36.683454595 1 redis-server (14098) &lt; gettimeofday
737661 21:46:36.683455753 1 redis-server (14098) > stat
737666 21:46:36.683464299 1 redis-server (14098) &lt; stat res=0 path=/etc/localtime
737670 21:46:36.683468715 1 redis-server (14098) > write fd=1(p) size=100
737673 21:46:36.683472956 1 redis-server (14098) > switch next=31412(docker) pgft_maj=0 pgft_min=5453 vm_size=20808 vm_rss=1776 vm_swap=0
737696 21:46:36.683507708 1 redis-server (14098) &lt; write res=100 data=1:M 30 Mar 21:46:36.683 # Creating Server TCP listening socket *:6379: bind: Add
737703 21:46:36.683519356 1 redis-server (14098) > exit_group
737706 21:46:36.683527569 1 redis-server (14098) > switch next=32724(docker) pgft_maj=0 pgft_min=5454 vm_size=20808 vm_rss=1932 vm_swap=0
737740 21:46:36.683605073 1 redis-server (14098) > switch next=31264(docker) pgft_maj=0 pgft_min=5454 vm_size=0 vm_rss=0 vm_swap=0
&lt;/strong><strong>737765 21:46:36.683686210 1 redis-server (14098) > procexit status=256</strong></pre>

Of course, it turns out to be the developer’s redis (the one in mesos container):

<pre># netstat -tulpn | grep :6379
tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN 
9423/redis-server *
tcp6       0      0 :::6379                 :::*                    LISTEN
9423/redis-server *
</pre>

We can double-check and verify using csysdig and navigating to Containers view:

[embed width="600" height="350"]https://sysdigrp2rs.wpengine.com/wp-content/uploads/2016/04/Mesos-blog-2.mp4[/embed]

It is worth mentioning that, while csysdig supports Mesos and Marathon filtering as well as preconfigured views, those would not be of help in this case - the failing redis container never really becomes a Mesos Task and, subsequently, is never added to Marathon App (which is the reason for blinking appearance in Sysdig Monitor UI). For that reason, we had to “dig” directly into the container level in order to see the redis container repeatedly recreated. Otherwise, Mesos/Marathon - specific views are readily available in csysdig:

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/04/mesos-img-2.png"></a>[![csysdig][6]][6] 

As with other features, Mesos/Marathon metadata and metrics are recorded in Sysdig Monitor, or you can manually capture data using open source sysdig, for later replay and filtering of the system activity.

### How to Enable Mesos/Marathon Support in Sysdig Tools To enable mesos support in sysdig or csysdig, use:

  
<pre>-m &lt;url [,marathon_url]>&lt;/url></pre> or: 

  
<pre>--mesos-api=&lt;url [,marathon_url]>&lt;/url></pre>

  
For example:  
<pre># csysdig -m http://10.10.10.10:5050, http://10.10.10.10:8080</pre>

  
Additionally, sysdig supports preconfigured filtering for mesos events, for example the following command:  
<pre># sysdig -m http://10.10.10.10:5050, http://10.10.10.10:8080 -pm mesos.task.id!=''</pre>

  
Results in formatted and filtered Mesos output formatted using predefined sysdig format for Mesos:  
<pre>%evt.num %evt.outputtime %evt.cpu %mesos.task.name (%container.id) %proc.name\ 
(%thread.tid:%thread.vtid) %evt.dir %evt.type %evt.info</pre>

  
In Sysdig Monitor Agent, there is no configuration needed to detect Mesos when at least one Agent runs on the same host where Mesos/Marathon API servers are running - the mesos-master process will auto-detected and connection will be attempted on ports 5050/8080 for Mesos/Marathon respectively. Configuration details are provided in the [User Guide][7].

A couple of facts are worth noting here:

Sysdig Monitor Agent recognizes and matches the Mesos/Marathon managed containers by receiving cluster information from API server and containers information from the cluster nodes themselves; this means that all managed nodes must run Sysdig Monitor Agent in order for cluster to be fully recognized.

Sysdig Monitor Agent will follow Mesos/Marathon leader migration “automagically” - no configuration needed. However, when API server is specified in the configuration file, automatic leader following must be explicitly enabled. See the above mentioned User Guide for more details.

### Conclusion

By adding support for Mesos, Marathon and DCOS, Sysdig provides a powerful tool for cluster monitoring and troubleshooting. Cluster status can be seen in real-time, using Sysdig Monitor or Sysdig open-source console- and ncurses-based tools; recording and replay capabilities provide further help for diagnosing problems that occurred in the past.

Sysdig recognizes any mesos framework, and its metadata will be used for aggregating data into logical tasks and applications.. At this time, framework-specific metadata (and, consequently, framework-specific cluster config/layout) is only available for Marathon framework, but we plan to add more support as our customers need it.

[Try it out][8] and let us know what you think!

 [1]: https://sysdigrp2rs.wpengine.com/press-releases/sysdig-mesos-partnership
 [2]: http://mesos.apache.org/
 [3]: https://mesosphere.github.io/marathon/
 [4]: /wp-content/uploads/2016/04/mesos-img-1.png
 [5]: https://sysdigrp2rs.wpengine.com/blog/no-plugins-required-application-visibility-inside-containers/
 [6]: /wp-content/uploads/2016/04/mesos-img-2.png
 [7]: http://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/207886103-Sysdig-Cloud-Agent-Mesos-Marathon
 [8]: https://sysdigrp2rs.wpengine.com