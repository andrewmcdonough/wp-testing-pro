---
ID: 5589
post_title: >
  Runtime Security for Kubernetes with
  Sysdig Falco
author: Michael Ducy
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/runtime-security-kubernetes-sysdig-falco/
published: true
post_date: 2017-12-21 13:18:36
---
The power of Kubernetes is how easy it is to quickly deploy and scale containerized applications. Developers can quickly package their application in a container, and then deploy it with just a few lines of YAML. However, this model requires for you to rethink how you handle container runtime security. How do you know what is running in that prepackaged container? How do you monitor the security of this container and watch for malicious activity?

Sysdig Falco is an open source, container security monitor designed to detect anomalous activity in your containers. Sysdig Falco taps into your host’s (or Node’s in the case Kubernetes) system calls to generate an event stream of all system activity. Falco’s rules engine then allows you to create rules based on this event stream, allowing you to alert on system events that seem abnormal. Since containers should have a very limited scope in what they run, you can easily create rules to alert on abnormal behavior inside a container.

Falco provides <a href="https://github.com/draios/falco/tree/dev/rules" target="_blank">rules for common</a> antipatterns such as:

*   Spawning a shell in a container

*   Config files being written in /etc 

*   Binaries changing

*   Package management running

## Running Falco as a Daemon Set {#runningfalcoasadaemonset}

*Note: You can find the code for the below example in the <a href="https://github.com/draios/falco/tree/dev/examples/k8s-using-daemonset" target="_blank">Falco GitHub repository</a>.*

Falco can be ran as a Daemon Set on Kubernetes to monitor any containers running. You can find examples of how to run a Falco Daemon Set in the project’s GitHub repo. The first thing you’ll need to do is create the service account and grant the account the appropriate permissions. The Falco pods will use this service account to authenticate with the Kubernetes API. With this integration to the Kubernetes API, Falco allows you to include Kubernetes specific data (Pod name, Node name, etc) in any alerts Falco generates.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/falco-service-account.gif" alt="Creating a Service Account for Falco"  width="1024" height="606" class="alignnone size-medium wp-image-5595" />][1] 
Once the service account is created, we can create a ConfigMap to store Falco’s rules and configuration. While the Falco Docker image could be ran without the ConfigMap, you will most likely want to modify the configuration, and the ConfigMap provides a convenient place to store your custom configuration. For example, you can modify the `falco.yaml` config to send a message to a Slack webhook.

    program_output:
      enabled: true
      keep_alive: false
      program: "jq '{text: .output}' | curl -d @- -X POST https://hooks.slack.com/services/see_your_slack_team/apps_settings_for/a_webhook_url"

[tweet_box design="default" float="none"]Runtime Security for Kubernetes with Sysdig Falco[/tweet_box] 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/falco-configmap.gif" alt="Creating a ConfigMap for Falco" width="1024" height="608" class="aligncenter size-medium wp-image-5594" />][2]

Now that we laid down the accounts and config Falco needs, we can create the Daemon Set. The Daemon Set will run a Pod on every Kubernetes Node in order to create an event stream from the system calls of that host. Each Pod will also pull down the configuration we created with the ConfigMap.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/falco-daemonset.gif" alt="Creating a Daemon Set for Falco" width="1024" height="607" class="aligncenter size-medium wp-image-5593" />][3]

Now that our Daemon Set is deployed, any behavior seen as abnormal will notify our Slack channel. For example, if we do a `kubectl exec` to a Pod and open a `bash` shell, Falco will send a message to Slack.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/Screen-Shot-2017-12-19-at-10.48.21-PM-1024x329.png" alt="Falco Events in Slack" width="1024" height="329" class="aligncenter size-medium wp-image-5596" />][4]

You can also use the Falco Event Generator to test your Falco setup. You can read more about the <a href="https://github.com/draios/falco/wiki/Generating-Sample-Events" target="_blank">event generator in Falco’s documentation</a>. The event generator is available as a Docker container, and we provide a <a href="https://github.com/draios/falco/tree/dev/examples/k8s-using-daemonset" target="_blank">Kubernetes Deployment</a> to deploy the event generator to a Kubernetes Cluster.

Sysdig Falco provides an easy way to alert when a container behaves outside what is considered normal. You can easily extend the <a href="https://github.com/draios/falco/blob/dev/rules/falco_rules.yaml" target="_blank">Falco rule set</a> to take into account scenarios specific to your environment. Refer to <a href="https://github.com/draios/falco/wiki/Falco-Rules" target="_blank">the Falco documentation</a> to learn more about writing your own rules. There’s also example <a href="https://github.com/draios/falco/blob/dev/rules/application_rules.yaml" target="_blank">rules for specific applications</a> such as MongoDB, Elastic Search, and Kafka. 

## Sysdig Secure {#sysdigsecure}

If you like the functionality of Sysdig Falco, but want more integrations, user activity auditing, the ability to kill or pause containers based on rules, and the ability to capture the state of a container when a rule fails check out Sysdig Secure. You can request a <a href="https://go.sysdigrp2rs.wpengine.com/docker-security-demo" target="_blank">customized demo</a>, or <a href="https://go.sysdigrp2rs.wpengine.com/secure-trial-request" target="_blank">kick off a trial</a>.

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/falco-service-account.gif
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/falco-configmap.gif
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/falco-daemonset.gif
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/12/Screen-Shot-2017-12-19-at-10.48.21-PM.png