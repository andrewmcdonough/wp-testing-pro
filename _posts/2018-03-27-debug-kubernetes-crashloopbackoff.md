---
ID: 6819
post_title: >
  What is a CrashLoopBackOff? How to
  alert, debug / troubleshoot, and fix
  Kubernetes CrashLoopBackOff events.
author: Knox Anderson
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/debug-kubernetes-crashloopbackoff/
published: true
post_date: 2018-03-27 12:52:13
---
In this blog we're going to talk about how to visualize, alert, and debug / troubleshoot a Kubernetes CrashLoopBackOff event. As all veteran Kubernetes users know, Kubernetes CrashLoopBackOff events are a way of life. It's happened to all of us at least once and normally we're stuck scratching our heads with no access to troubleshooting tools inside the container.

Are you in a hurry: jump directly into [How to debug / troubleshoot and fix Kubernetes CrashLoopBackOff][1]!

## What is a Kubernetes CrashLoopBackOff? The meaning. {#whatisakubernetescrashloopbackoffthemeaning}

A CrashloopBackOff means that you have a pod starting, crashing, starting again, and then crashing again. 

A `PodSpec` has a `restartPolicy` field with possible values `Always`, `OnFailure`, and `Never` which applies to all containers in a pod. The default value is `Always` and the `restartPolicy` only refers to restarts of the containers by the kubelet on the same node (so the restart count will reset if the pod is rescheduled in a different node). Failed containers that are restarted by the kubelet are restarted with an exponential back-off delay (10s, 20s, 40s …) capped at five minutes, and is reset after ten minutes of successful execution. This is an example of a `PodSpec` with the `restartPolicy` field:

    apiVersion: v1
    kind: Pod
    metadata:
      name: dummy-pod
    spec:
      containers:
        - name: dummy-pod
          image: ubuntu
      restartPolicy: Always

## Why does a CrashLoopBackOff occur? {#whydoesacrashloopbackoffoccur}

A quick Google search will show us that crash loop events can happen for a number of different reasons (and they happen frequently). Here are some of the umbrella causes for why they occur:

*   The application inside the container keeps crashing
*   <a href="https://stackoverflow.com/questions/41604499/my-kubernetes-pods-keep-crashing-with-crashloopbackoff-but-i-cant-find-any-lo" target="_blank">Some type of parameters of the pod or container have been configured incorrectly</a>
*   <a href="https://stackoverflow.com/questions/35537834/debugging-a-container-in-a-crash-loop-on-kubernetes" target="_blank">An error has been made when deploying Kubernetes</a> 

## How can I see if there are CrashLoopBackOff in my cluster? {#howcaniseeiftherearecrashloopbackoffinmycluster}

Run your standard `kubectl get pods` command and you'll be able to see the status of any pod that is currently in CrashLoopBackOff:

    kubectl get pods --namespace nginx-crashloop
    NAME                     READY     STATUS             RESTARTS   AGE
    flask-7996469c47-d7zl2   1/1       Running            1          77d
    flask-7996469c47-tdr2n   1/1       Running            0          77d
    nginx-5796d5bc7c-2jdr5   0/1       CrashLoopBackOff   2          1m
    nginx-5796d5bc7c-xsl6p   0/1       CrashLoopBackOff   2          1m

Actually if you see pods in Error status, probably they will get into CrashLoopBackOff soon:

    kubectl get pods --namespace nginx-crashloop
    NAME                     READY     STATUS    RESTARTS   AGE
    flask-7996469c47-d7zl2   1/1       Running   1          77d
    flask-7996469c47-tdr2n   1/1       Running   0          77d
    nginx-5796d5bc7c-2jdr5   0/1       Error     0          24s
    nginx-5796d5bc7c-xsl6p   0/1       Error     0          24s

Doing a `kubectl describe pod` will give us more information on that pod:

    kubectl describe pod nginx-5796d5bc7c-xsl6p --namespace nginx-crashloop
    Name:           nginx-5796d5bc7c-xsl6p
    Namespace:      nginx-crashloop
    Node:           ip-10-0-9-132.us-east-2.compute.internal/10.0.9.132
    Start Time:     Tue, 27 Mar 2018 19:11:05 +0200
    Labels:         app=nginx-crashloop
                    name=nginx
                    pod-template-hash=1352816737
                    role=app
    Annotations:    kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"nginx-crashloop","name":"nginx-5796d5bc7c","uid":"fb9e9518-f542-11e7-a8f2-065cff0...
    Status:         Running
    IP:             10.47.0.15
    Controlled By:  ReplicaSet/nginx-5796d5bc7c
    Containers:
      nginx:
        Container ID:   docker://513cab3de8be8754d054a4eff45e291d33b63e11b2143d0ff782dccc286ba05e
        Image:          nginx
        Image ID:       docker-pullable://nginx@sha256:c4ee0ecb376636258447e1d8effb56c09c75fe7acf756bf7c13efadf38aa0aca
        Port:           <none>
        State:          Waiting
          Reason:       CrashLoopBackOff
        Last State:     Terminated
          Reason:       Error
          Exit Code:    1
          Started:      Tue, 27 Mar 2018 19:13:15 +0200
          Finished:     Tue, 27 Mar 2018 19:13:16 +0200
        Ready:          False
        Restart Count:  4
        Environment:    <none>
        Mounts:
          /etc/nginx/nginx.conf from config (rw)
          /var/run/secrets/kubernetes.io/serviceaccount from default-token-chcxn (ro)
    Conditions:
      Type           Status
      Initialized    True 
      Ready          False 
      PodScheduled   True 
    Volumes:
      config:
        Type:      ConfigMap (a volume populated by a ConfigMap)
        Name:      nginxconfig
        Optional:  false
      default-token-chcxn:
        Type:        Secret (a volume populated by a Secret)
        SecretName:  default-token-chcxn
        Optional:    false
    QoS Class:       BestEffort
    Node-Selectors:  nginxcrash=allowed
    Tolerations:     node.alpha.kubernetes.io/notReady:NoExecute for 300s
                     node.alpha.kubernetes.io/unreachable:NoExecute for 300s
    Events:
      Type     Reason                 Age               From                                               Message
      ----     ------                 ----              ----                                               -------
      Normal   Scheduled              2m                default-scheduler                                  Successfully assigned nginx-5796d5bc7c-xsl6p to ip-10-0-9-132.us-east-2.compute.internal
      Normal   SuccessfulMountVolume  2m                kubelet, ip-10-0-9-132.us-east-2.compute.internal  MountVolume.SetUp succeeded for volume "config"
      Normal   SuccessfulMountVolume  2m                kubelet, ip-10-0-9-132.us-east-2.compute.internal  MountVolume.SetUp succeeded for volume "default-token-chcxn"
      Normal   Pulled                 1m (x3 over 2m)   kubelet, ip-10-0-9-132.us-east-2.compute.internal  Successfully pulled image "nginx"
      Normal   Created                1m (x3 over 2m)   kubelet, ip-10-0-9-132.us-east-2.compute.internal  Created container
      Normal   Started                1m (x3 over 2m)   kubelet, ip-10-0-9-132.us-east-2.compute.internal  Started container
      Warning  BackOff                1m (x5 over 1m)   kubelet, ip-10-0-9-132.us-east-2.compute.internal  Back-off restarting failed container
      Warning  FailedSync             1m (x5 over 1m)   kubelet, ip-10-0-9-132.us-east-2.compute.internal  Error syncing pod
      Normal   Pulling                57s (x4 over 2m)  kubelet, ip-10-0-9-132.us-east-2.compute.internal  pulling image "nginx"

[tweet_box design="default" float="none"]How to alert, troubleshoot and fix #Kubernetes CrashLoopBackOff events[/tweet_box] 

### Visualizing Kubernetes events in Sysdig Monitor {#visualizingkuberneteseventsinsysdigmonitor}

CrashLoopBackOff events can be viewed through <a href="https://sysdigrp2rs.wpengine.com/product/monitor/" target="_blank">Sysdig Monitor</a> on the events tab. Sysdig Monitor will natively ingest both Kubernetes and <a href="https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/210530163-Event-integrations-Application-Events#Docker" target="_blank">Docker events</a> for users alert on, and overlay on charts of the system performance.

### Kubernetes Events Ingested by Sysdig {#kuberneteseventsingestedbysysdig}

    kubernetes:
        node:
          - TerminatedAllPods       # Terminated All Pods      (information)
          - RegisteredNode          # Node Registered          (information)*
          - RemovingNode            # Removing Node            (information)*
          - DeletingNode            # Deleting Node            (information)*
          - DeletingAllPods         # Deleting All Pods        (information)
          - TerminatingEvictedPod   # Terminating Evicted Pod  (information)*
          - NodeReady               # Node Ready               (information)*
          - NodeNotReady            # Node not Ready           (information)*
          - NodeSchedulable         # Node is Schedulable      (information)*
          - NodeNotSchedulable      # Node is not Schedulable  (information)*
          - CIDRNotAvailable        # CIDR not Available       (information)*
          - CIDRAssignmentFailed    # CIDR Assignment Failed   (information)*
          - Starting                # Starting Kubelet         (information)*
          - KubeletSetupFailed      # Kubelet Setup Failed     (warning)*
          - FailedMount             # Volume Mount Failed      (warning)*
          - NodeSelectorMismatching # Node Selector Mismatch   (warning)*
          - InsufficientFreeCPU     # Insufficient Free CPU    (warning)*
          - InsufficientFreeMemory  # Insufficient Free Mem    (warning)*
          - OutOfDisk               # Out of Disk              (information)*
          - HostNetworkNotSupported # Host Ntw not Supported   (warning)*
          - NilShaper               # Undefined Shaper         (warning)*
          - Rebooted                # Node Rebooted            (warning)*
          - NodeHasSufficientDisk   # Node Has Sufficient Disk (information)*
          - NodeOutOfDisk           # Node Out of Disk Space   (information)*
          - InvalidDiskCapacity     # Invalid Disk Capacity    (warning)*
          - FreeDiskSpaceFailed     # Free Disk Space Failed   (warning)*
        pod:
          - Pulling           # Pulling Container Image          (information)
          - Pulled            # Ctr Img Pulled                   (information)
          - Failed            # Ctr Img Pull/Create/Start Fail   (warning)*
          - InspectFailed     # Ctr Img Inspect Failed           (warning)*
          - ErrImageNeverPull # Ctr Img NeverPull Policy Violate (warning)*
          - BackOff           # Back Off Ctr Start, Image Pull   (warning)
          - Created           # Container Created                (information)
          - Started           # Container Started                (information)
          - Killing           # Killing Container                (information)*
          - Unhealthy         # Container Unhealthy              (warning)
          - FailedSync        # Pod Sync Failed                  (warning)
          - FailedValidation  # Failed Pod Config Validation     (warning)
          - OutOfDisk         # Out of Disk                      (information)*
          - HostPortConflict  # Host/Port Conflict               (warning)*
        replicationController:
          - SuccessfulCreate    # Pod Created        (information)*
          - FailedCreate        # Pod Create Failed  (warning)*
          - SuccessfulDelete    # Pod Deleted        (information)*
          - FailedDelete        # Pod Delete Failed  (warning)*

Custom events can be sent into the Sysdig Monitor events API to be used for correlation and alerting as well. For example you can send a custom event when you run a new deployment from Jenkins, when you do a roll-back of a broken version or when your cloud infrastructure changes.

The custom events tab of Sysdig Monitor gives us a feed of all events that have happened across my distributed Kubernetes environment. Here we can see the timestamp, event name, description, severity and other details. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/kubernetes_events_crashloopbackoff-1024x275.png" alt="CrashLoopBackOff events" width="1024" height="275" class="aligncenter size-large wp-image-6823" />][2]

Clicking on an individual event brings up further details about that specific event and more granular details about where it occurred in our infrastructure. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/crashloopbackoff_event_details-1024x166.png" alt="CrashLoopBackOff event details" width="1024" height="166" class="aligncenter size-large wp-image-6824" />][3]

We can also correlate these events with the behavior of our systems. Looking at the image below we can quickly see when a specific backoff event occurred and if it caused and change to the performance of the system. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/crashloopbackoff_event_in_graph.png" alt="CrashLoopBackOff event metric correlation" width="656" height="607" class="aligncenter size-full wp-image-6825" />][4]

## How to alert on Kubernetes CrashLoopBackOff {#howtoalertonkubernetescrashloopbackoff}

For alerting purposes we'll want to use the metric `kubernetes.pod.restart.count`. This will give us the ability to do analysis on the trend of restart counts over time, and promptly notify our team of any anomalies.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/crashloopbackoff_pod_restart_count_alert.png" alt="CrashLoopBackOff pod restart metric alert" width="685" height="954" class="aligncenter size-full wp-image-6826" />][5]

Depending on the delay in your environment you'll want to toggle the time settings. This alert is configured to trigger if any pod restarts more than 4 times over a two minute span, which is usually an indicator of a CrashLoopBackOff event. This alert is one of the default alerts for Kubernetes environments.

Enabling a Sysdig Capture is also very important for the troubleshooting of a CrashLoopBackOff. A Sysdig capture is a full recording of everything that happened on the system at the point in time when an alert triggered. Captures can be opened with Sysdig Inspect for deep forensic and troubleshooting analysis so teams can respond and recover from incidents quicker.

In a similar fashion, you can also configure a CrashLoopBackOff alert based on the events that Sysdig collects:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/crashloopbackoff_event_alert.png" alt="CrashLoopBackOff event alert" width="686" height="1012" class="aligncenter size-full wp-image-6827" />][6]

## How to debug / troubleshoot and fix Kubernetes CrashLoopBackOff {#howtodebugtroubleshootandfixkubernetescrashloopbackoff}

You can manually <a href="https://sysdigrp2rs.wpengine.com/blog/troubleshooting-containers-when-theyre-gone/" target="_blank">trigger a Sysdig capture</a> at any point in time by selecting the host where you see the CrashLoopBackOff is occurring and starting the capture. You can take it manually with <a href="https://sysdigrp2rs.wpengine.com/opensource/" target="_blank">Sysdig open source</a> if you have it installed on that host. But here will take advantage of the Sysdig Monitor capabilities that can automatically take this capture file as a response to an alert, in this case a CrashLoopBackOff alert.

The first troubleshooting action item is to open the capture file that was recorded at the point in time that the event was happening on the host. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/crashloopbackoff_troubleshooting_captures-1024x130.png" alt="CrashLoopBackOff debug Sysdig capture" width="1024" height="130" class="aligncenter size-large wp-image-6828" />][7]

When a capture is opened in Sysdig Monitor a browser window will pop up with <a href="https://sysdigrp2rs.wpengine.com/blog/sysdig-inspect-explained-visually/" target="_blank">Sysdig Inspect</a>. Inspect allows you to do system call analysis through a GUI for more efficient correlation and troubleshooting analysis. Within the scope of our <a href="https://sysdigrp2rs.wpengine.com/product/secure/" target="_blank">Sysdig Secure</a>, our container run-time security product, Sysdig Inspect is used for post-mortem analysis and forensics.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/crashloopbackoff_debug1-1024x529.png" alt="CrashLoopBackOff debug / troubleshooting" width="1024" height="529" class="aligncenter size-large wp-image-6829" />][8]

To troubleshoot this event we'll want to look at everything that is occurring the infrastructure column of Sysdig Inspect. Selecting the Docker Events tiles will bring those events into the timeline at the bottom.

Let's try to troubleshoot what's going on here. A good first step is to drill down into `Container Died Events`. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/crashloopbackoff_debug2-1024x529.png" alt="CrashLoopBackOff debug Docker died" width="1024" height="529" class="aligncenter size-large wp-image-6830" />][9]

OK, so it seems that the Nginx containers are having trouble. Looking at the timestamps they die shortly after being created. Let's drill down in any of the Nginx containers and there select Processes on the left hand side.

We know our Nginx container only executes one process "nginx" so from the Processes filter by `proc.name = nginx`.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/crashloopbackoff_debug3-1024x529.png" alt="CrashLoopBackOff debug crashing processes" width="1024" height="529" class="aligncenter size-large wp-image-6831" />][10]

Sysdig Inspect filters use the <a href="https://github.com/draios/sysdig/wiki/Sysdig-User-Guide#user-content-filtering" target="_blank">Sysdig open-source syntax</a> and can be used to pinpoint activity.

Edit example of blog post.

We can click on the Errors section, but nothing significant appears there, no failed system calls. Let's move into the Files section to inspect file system activity. There will see a `error.log` file, that probably has some information for us. We can see its I/O activity clicking on the `I/O Streams` icon.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/crashloopbackoff_debug4-1024x529.png" alt="CrashLoopBackOff debug access files" width="1024" height="529" class="aligncenter size-large wp-image-6832" />][11]

So from the content written in the `error.log` file until the container died, appears that Nginx cannot resolve a configured upstream server. We know why the Nginx fails, but can we look at what was the configured DNS server for that pod? Sure, just go back and get the streams for `resolv.conf`.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/crashloopbackoff_debug5-1024x529.png" alt="CrashLoopBackOff debug error.log" width="1024" height="529" class="aligncenter size-large wp-image-6833" />][12]

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/crashloopbackoff_debug6-1024x529.png" alt="CrashLoopBackOff debug resolv.conf" width="1024" height="529" class="aligncenter size-large wp-image-6834" />][13]

From here we can go even further and look at the DNS requests 53/UDP, and seems that the response was not found. This gives us further troubleshooting clues: we deployed the Nginx ReplicaSets first and the upstream Kubernetes service later. Nginx has a particularity, it caches the proxy names (like "flask") at startup time, not upon client request. In other words, we have deployed the different Kubernetes entities in the wrong dependency order.

## Conclusion {#conclusion}

While something like a pod restarting is an easy thing to spot, responding and recovering quickly from a potential degradation in a production service can be much harder, especially when the logs from the container are gone, you cannot reproduce the problem outside a specific environment or you just don't have the troubleshooting tools inside the container.

This is why further troubleshooting preparations like Sysdig captures are needed. They provide full container context and complete visibility to any interprocess communication, files written, and network activity. Like a time machine! Troubleshooting at the syscall level can be tricky but now with Sysdig Inspect it's a breeze!

 [1]: #howtodebugtroubleshootandfixkubernetescrashloopbackoff
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/kubernetes_events_crashloopbackoff.png
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/crashloopbackoff_event_details.png
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/crashloopbackoff_event_in_graph.png
 [5]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/crashloopbackoff_pod_restart_count_alert.png
 [6]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/crashloopbackoff_event_alert.png
 [7]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/crashloopbackoff_troubleshooting_captures.png
 [8]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/crashloopbackoff_debug1.png
 [9]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/crashloopbackoff_debug2.png
 [10]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/crashloopbackoff_debug3.png
 [11]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/crashloopbackoff_debug4.png
 [12]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/crashloopbackoff_debug5.png
 [13]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/crashloopbackoff_debug6.png
