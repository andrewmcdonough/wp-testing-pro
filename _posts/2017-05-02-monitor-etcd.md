---
ID: 4040
post_title: How to Monitor etcd on Kubernetes
author: Mateo Burillo
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/monitor-etcd/
published: true
post_date: 2017-05-02 14:08:20
---
In the following tutorial we'll show how to monitor etcd as a registry service for a Kubernetes cluster, including metrics, alerts, health check, performance and common failure points. But first things first, 

## What is etcd?  {#what-is-etcd-etcd-registry-a-kubernetes-component}

The motivation of <a href="https://coreos.com/etcd/docs/latest/getting-started-with-etcd.html" target="_blank"> etcd </a> is to provide a distributed key-value dynamic database which maintains a "configuration registry". This registry is one of the foundations of a Kubernetes cluster service directory, peer discovery and centralized configuration management. It bears certain resemblance to a Redis database, classical LDAP configuration backends or even the Windows Registry, if you are more familiar with those technologies. 

According to its developers, ** etcd ** aims to be: 

*   Simple: well-defined, user-facing API (JSON and gRPC) 
*   Secure: automatic TLS with optional client cert authentication 
*   Fast: benchmarked 10,000 writes/sec 
*   Reliable: properly distributed using Raft 

Kubernetes uses the etcd distributed database to store its REST API objects (under the `/registry` directory key): pods, secrets, daemonsets, deployments, namespaces, events, etc. 

    # etcdctl ls /registry
    /registry/pods
    /registry/events
    /registry/minions
    /registry/ranges
    /registry/replicasets
    /registry/serviceaccounts
    /registry/controllers
    /registry/namespaces
    /registry/configmaps
    /registry/services
    /registry/clusterrolebindings
    /registry/clusterroles
    /registry/secrets
    /registry/daemonsets
    /registry/deployments
    

[ Raft ][1] is a "consensus" algorithm, this is, a method to achieve value convergence over a distributed and fault-tolerant set of cluster nodes. Designed to overcome network errors, clock discrepancies, race conditions, complete node failures and attachment of new nodes. It abstracts away the core complexity of assembling a distributed system. 

Without going into the gory details that you will find in the referenced articles, the basics of what you need to know: 

*   Node status can be one of: Follower, Candidate (briefly), Leader 
*   If a Follower cannot locate the current Leader, it will become Candidate 
*   The voting system will elect a new Leader amongst the Candidates 
*   Registry value updates (commits) always go through the Leader 
*   Once the Leader has received the ack from the majority of Followers the new value is considered "committed" 
*   The cluster will survive as long as most of the nodes remain alive 

Maybe the most remarkable features of etcd are the straightforward way of accessing the service using REST-like HTTP calls, that makes integrating third party agents as simple as you can get, and its master-master protocol which automatically elects the cluster Leader and provides a fallback mechanism to switch this role if needed. 

## etcd cluster common points of failure  {#etcd-cluster-common-points-of-failure}

Most of the time your etcd cluster works so neatly, it is easy to forget its nodes are running. Keep in mind, however, that Kubernetes absolutely needs this registry to function, a major etcd failure will seriously cripple or even take down your container infrastructure. Pods currently running will continue to run, but you cannot make any further operations. When you re-connect etcd and Kubernetes again, state incoherences could cause additional malfunction. 

Cautionings aside, you can start by monitoring these three common issues in your cluster: 

*   [Node unavailable (Follower down and/or cluster Leader down)][2] 
*   [Entire etcd service failure][3] 
*   [Latency/network synchronization, lag between the Leader and Followers][4] 

## etcd monitoring and alerting  {#etcd-monitoring-and-alerting}

You can run etcd in Kubernetes, inside Docker containers or as an independent cluster (either in Docker containers or directly bare-metal). Usually, for simple scenarios, etcd is deployed in a Docker container like other Kubernetes services as the API server, the controller, scheduler or the kubelet. On more advanced scenarios etcd is often an external service, in these cases you will normally see 3 or more nodes to achieve the required redundancy. 

### etcd node availability  {#etcd-node-availability}

An obvious error scenario for any cluster is that you lose one of the nodes. The cluster will continue operating, but it is probably a good idea to receive an alert, diagnose and recover before you continue losing nodes and risk facing the next scenario, total service failure. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/figure14.png" width="978" height="619" class="alignleft size-full wp-image-4061" alt="basic alert" />][5] 

Tagging your monitoring agents provides a very convenient way of defining groups that can be later on used for your monitoring context. Only two agent tags are used in this scenario, the `
  etcd
 ` nodes and the `
  kubernetes
 ` nodes. The basic entity where you want to apply the alert test is defined in the `
  Segmented by
 ` field, thus, this alert will perform one verification per `host.mac` but we can also use `host.hostName`. The check is going to wait 5 minutes before considering it down, to avoid alerting on temporary network glitches. 

You can check the log of the etcd service on one of the remaining nodes at the failure time specified in the alert: 

    etcd3 etcd2[962]: failed to read 3f744bbb6c6ab1c3 on stream MsgApp v2 (read tcp 10.0.2.15:39894->
    etcd3 etcd2[962]: the connection with 3f744bbb6c6ab1c3 became inactive
    

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/figure2.png" width="514" height="210" class="alignleft size-full wp-image-4047" alt="rpc errors" />][6] 

Its peers have detected the follower failure and the cluster will continue working, but as you can see from the chart above, the cluster Leader is registering that all the RPC requests to this specific follower are suddenly failing. 

If the node which goes offline was actually the former cluster Leader, the Raft protocol will negotiate a new one: 

    etcd2 etcd2[958]: raft.node: 3f744bbb6c6ab1c3 <b>lost leader</b> 229b0bd139298d6b at term 1275
    etcd2 etcd2[958]: raft.node: 3f744bbb6c6ab1c3 <b>elected leader</b> 2d4437615e2062b3 at term 1275
    etcd2 etcd2[958]: failed to read 229b0bd139298d6b on stream MsgApp v2
    etcd2 etcd2[958]: the connection with 229b0bd139298d6b became inactive
    

Take into account that your cluster will stop working if you lose most of your nodes. 

**But, what happens if you dynamically add or remove etcd nodes?** 

This is totally supported by etcd, if you have a cluster with 3 nodes, add new 6 nodes and then 5 nodes go down, your cluster will become inoperative. You can explicitly remove an etcd node using a `member remove` command, in this case, the removed nodes won't be considered towards the total number of nodes needed for cluster coherency. Thus, if you <a href="https://ngineered.co.uk/blog/how-to-replace-a-etcd-node" target="_blank"> explicitly remove </a> 2 nodes in a 3 nodes scenario, the remaining node will continue working in a healthy state (but without HA), if these same 2 nodes go down unexpectedly, the cluster will fail. 

### etcd service unavailable  {#etcd-service-unavailable}

As mentioned before, losing the whole etcd registry means that Kubernetes cannot read/write values in the registry, leading to its malfunctioning or even sudden stop, this is an event that requires to be immediately addressed by the administrators, so you want to configure it as "Critical". 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/figure3.png" width="973" height="615" class="alignleft size-full wp-image-4048" alt="etcd failure" />][7] 

This alert will monitor all the hosts that contain an agent with the role `
  etcd
 ` and each one of the physical nodes (`Segmented by: host.mac`) will be treated as a different entity. 

Since the `
  Across
 ` field is set to `
  any
 ` , all the samples collected for every `
  host.mac
 ` over a minute will be evaluated separately, therefore, each one of the etcd nodes can trigger the alert independently. This could be useful, for instance, to detect a network failure that has left one of the nodes isolated. 

If you select `
  every
 ` instead, the alert will only fire if all of groups fulfill the condition simultaneously. This approach may be more accurate if what you want to detect is a complete etcd cluster failure. 

Additionally, you can configure a somewhat more complete check: 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/figure15.png" width="976" height="618" class="alignleft size-full wp-image-4062" alt="etcd failure from k8s" />][8] 

In this case, the alert will check that each one of the pods can connect to an etcd host/port. To avoid firing the alarm from development/test scenarios you can make use of <a href="https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/" target="_blank"> Kubernetes namespaces </a> and restrict the check to production pods. 

For this test, you need to customize <a href="https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/205147903-Metrics-integrations-Application-Checks" target="_blank"> (see Sysdig Monitor docs) </a> the etcd check, providing the target host and port. 

<script src="https://gist.github.com/8f6df790dae008396984bc419f084906.js"></script> <noscript>
  <pre><code>
File: dragent.yaml
------------------

customerid: *************************************
tags: role:kubernetes
app_checks:
 - name: etcd
   pattern:
     comm: init
   conf:
     url: "http://172.17.4.51:2379/"
</code></pre>
</noscript>

### Cluster member Latency  {#cluster-member-latency}

Another metric to consider is the member latency: delay until a Follower achieves data coherence with the cluster Leader. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/figure5.png" width="1075" height="443" class="alignleft size-full wp-image-4050" alt="latencies" />][9] 

In a testing scenario running on a local network, the sync latencies are fairly stable around 0.01-0.02 sec. 

This is how we set an alert when this latency goes higher than 0.1 sec for any of the members, possibly signaling a network clog, package loss, etc: 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/figure16.png" width="974" height="576" class="alignleft size-full wp-image-4063" alt="latency alert" />][10] 

Now if we simulate a network jam in the internal LAN which interconnects the current etcd Leader with one of it's Followers, will see something like this: 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/figure7.png" width="516" height="212" class="alignleft size-full wp-image-4052" alt="leader latency" />][11] 
The "Leader Latency by Follower" graph that you can find accessing the etcd metrics evidence the synchronization lag problems. 

Sudden short bursts of latency can show up and disappear without any intervention, to avoid a noisy alert that does not require any specific actions, you would rather configure the alert to trigger only if the high latencies are sustained for example over 10+ minutes, as opposed to `
    at least once
   ` used in the example . 

## Monitoring etcd performance  {#monitoring-etcd-performance}

From the "Explore" tab in Sysdig Monitor, you can filter the metrics by the string "etcd" in the lower left panel and look through all the etcd views and metrics available. You can choose the ones more relevant to your use case and compose a custom board. 

Conveniently enough, if you go to "Dashboards" tab, and add a new dashboard, there is already an "etcd" preconfigured one where you can check the most useful metrics: 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/figure9.png" alt="etcd dash" width="1025" height="486" class="alignleft size-full wp-image-4056" />][12] 

To simulate a high load scenario, let's reproduce a common situation: you need to upgrade the software and/or hardware of one of the Kubernetes nodes. Moving all the containers away and later on re-filling the node again will generate a considerable amount of stress in the etcd registry. 

Taking a quick look at the etcd dashboard, you realize that the "idle" state for this cluster is around 40 atomic CompareAndSwap cluster operations per second: 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/figure10.png" alt="compareandswap" width="1068" height="441" class="alignleft size-full wp-image-4057" />][13] 

Let's configure a load warning if this metric reaches 100+ operations per second (on average): 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/figure17.png" alt="100 ops" width="994" height="545" class="alignleft size-full wp-image-4064" />][14] 

According to the "Overview" table, the cluster is already withstanding considerable loads: 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/figure12.png" alt="cluster load" width="1299" height="331" class="alignleft size-full wp-image-4059" />][15] 

Now, let's proceed to drain one of the Kubernetes workers, forcing a cluster re-balance: 

    $ kubectl drain --grace-period=10 kubeworker2
    

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/figure13.png" alt="cluster load peak" width="1071" height="446" class="alignleft size-full wp-image-4060" />][16] 

If you check your etcd dashboard, there is a noticeable peak during this operation, more than enough to trigger the warning message. Un-cordoning the node will cause a similar peak while the pods return to the refreshed node: 

    $ kubectl uncordon kubeworker2
    

## Additional etcd metrics  {#additional-etcd-metrics}

The default Sysdig Monitor dashboard for etcd will show you the most relevant metrics and the graphs are a nice intuitive place to start. But there is more to it, some metrics are not included on the default view but you can add the ones that might be interested in. 

To name a few additional metrics: 

*   Standard Deviation `
     etcd.leader.latency.stddev
    ` : If you are expecting high loads of registry updates, for example during a node cordon/uncordon, maybe an etcd member showing higher standard deviations is more relevant than the absolute load value per se. 
*   Bandwidth and packets per second `
     etcd.self.send.pkgrate, etcd.self.recv.pkgrate
    ` : Additional to the number of successful / failed operations shown in the graphs, you can also get a measure of synchronization packages exchanged between the Leader and Followers, this could be useful to design network capacity requirements. 
*   Service availability with `
     etcd.can_connect
    ` : This metric was used already in an alert above. Sysdig Monitor agent checks service availability, simple but very useful. 

Eventually, in a complex enough deployment, you are going to wish for a metric rather specific to your scenario. To continue with the article's context, imagine that you want a metric and alerts for the etcd database size: 

    etcd1 etcd2 # pwd
    /var/lib/etcd2
    etcd1 etcd2 # du -csh
    324M    .
    324M    total
    

If you know which commands to use and the specific output you wish to monitor, you can always write your <a href="https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/208032026-Metrics-integrations-Build-a-Custom-App-Check" target="_blank"> own custom checks </a> and run them within the Sysdig Monitor agent. 

## Conclusions  {#conclusions}

etcd is a simple and robust service which is required to deploy a Kubernetes cluster. Even though the Raft distributed consensus algorithm is able to overcome most of the temporal network failures, node losses, cluster splits, etc, if you are running Kubernetes in production, it is essential to monitor and set up alerts on relevant cluster events before it's too late. 

If you dig in the list of <a href="https://github.com/coreos/etcd/blob/master/error/error.go" target="_blank"> etcd error codes </a> , there are of course more advanced cases that the ones covered in this brief article: max numbers of peers in the cluster, anomaly detection between the Kubernetes and etcd nodes, Raft internal errors, registry size, etc, to name a few, although we leave those for a second part of this article in the future. 

Monitoring etcd with Sysdig Monitor its really easy, just one tool to monitor etcd and Kubernetes. Sysdig Monitor agent will collect all the etcd metrics and you can quickly setup the most important etcd alerts. If you haven't tried Sysdig Monitor yet, you are just one click away from our <a href="https://sysdigrp2rs.wpengine.com/sign-up/" target="_blank"> free Trial! </a>

 [1]: https://raft.github.io/
 [2]: #etcd-node-availability
 [3]: #etcd-service-unavailable
 [4]: #cluster-member-latency
 [5]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/figure14.png
 [6]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/figure2.png
 [7]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/figure3.png
 [8]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/figure15.png
 [9]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/figure5.png
 [10]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/figure16.png
 [11]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/figure7.png
 [12]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/figure9.png
 [13]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/figure10.png
 [14]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/figure17.png
 [15]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/figure12.png
 [16]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/04/figure13.png