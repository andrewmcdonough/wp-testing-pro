---
ID: 7723
post_title: 'Runtime container security &#8211; How to implement open source container security (part 1).'
author: Mateo Burillo
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/oss-container-security-runtime/
published: true
post_date: 2018-07-12 14:36:55
---
Container security is top-of-mind for any organization adopting Docker and Kubernetes, and this open source security guide is a comprehensive resource for anyone who wants to learn how to implement a complete open source container security stack for Docker and Kubernetes.

Securing a container platform is a multi-step process spanning from development to production.

There are <a href="https://sysdigrp2rs.wpengine.com/blog/7-docker-security-vulnerabilities/" target="_blacnk">Docker security best practices</a> for packaging apps as containers. Security must be implemented at the Infrastructure layer, including host, Docker and Kubernetes. Our <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-guide/" target="_blank">Kubernetes Security Guide</a> goes through how to secure Kubernetes cluster, components and use Kubernetes features to increase security in your apps, like RBAC, TLS, Pod Security Policies or Network Policies.

But additional security gaps remain, such as:

1.  **Image vulnerabilities**. Either if you are building images for your apps or you are using unmodified third party images, you need to track down any known vulnerability present in those images. This is known as Docker image scanning. Our second part will show <a href="https://sysdigrp2rs.wpengine.com/blog/container-security-docker-image-scanning/" target="_blank">how to implement open source Docker scanning</a>.
2.  **Runtime container security**. Production is the longest period of a container lifecycle, what happens after deployment? Is your container doing what is supposed to do? This is going to be out first part, how to implement runtime container security with open source tools.

Read on for helpful information and how-to tips on:

[
*   Runtime Container Security (what it is and how to implement it)][1] 
[
*   Understanding Response Engine and Security Playbooks][2] 
[
*   Runtime Container Security Open Source Tools][3] 
[
*   Building a Kubernetes Runtime Security Stack with Open Source Tools
    ][4] 
[
*   Gain Visibility Inside your Containers and Applications with Sysdig Falco (installation steps included)][5] 
[
*   Define your Runtime Container Security Policy for Applications, Services and the Cluster with Falco Default Ruleset Library][6] 
[
*   Kubernetes Incident Response
    ][7] 
[
*   Deploying Kubernetes Response Engine Components: NATS and Kubeless framework][8] 
[
*   Deploying Kubernetes Security Playbooks on Kubeless
    ][9] 
[
*   Slack Webhook Notification][10] 
[
*   Delete Offending Pod][11] 
[
*   Taint a Node and Stop Scheduling][12] 
[
*   Pod Network Isolation][13]

[
*   Kubernetes Security Logging with Falco & Fluentd][14] 
[tweet_box design="default" float="none"]Using @Sysdig #Falco, @nats_io and @Bitnami Kubeless FaaS to deploy a complete #container #Kubernetes #security opensource stack.[/tweet_box] 

## Runtime Container Security {#runtimesecurity}

Runtime security is a part of everything that happens once a container and the processes inside it are running.

If a container is not running as expected, could be an attack that exploited a well known vulnerability -and this can be potentially catched with container image scanning- but there are other possible reasons outside this approach. We need runtime container security to protect from:

*   Bogus configuration, intentional or not, leading to data losses, security intrusion and eventually information disclosure.
*   0-day vulnerabilities or vulnerabilities in your own software.
*   Weak or leaked credentials, keys and other sensitive information that might allow remote access.
*   Resource abuse for cryptocurrency mining or just DoS. We wrote about this in: "<a href="https://sysdigrp2rs.wpengine.com/blog/detecting-cryptojacking/" target="_blank">Fishing for Miners – Cryptojacking Honeypots in Kubernetes</a>".

### Implementing Runtime Container Security {#implementingruntimesecurity}

Runtime container security can be implemented through monitoring the behavior at execution time. Containers are intended to be simple (typically a single process with only the required dependencies and libraries, and often as part of a microservices app, so performing a single task), thus, it is relatively easy to define a behavior pattern through strict whitelists that detect any action that deviates from the norm:

*   File directories and paths that can be accessed and/or written.
*   Binaries that should be running.
*   External services that should be contacted and in general network connections.
*   System calls that can be executed.

Indicators of compromise can be also identified through blacklists, patterns of behavior that we know should never happen, like:

*   Writing files in binary paths. Containers are meant to be immutable and in-place upgrades should never happen.
*   Writing files in configuration paths like `/etc`. For the same reasons we shouldn't reconfigure services while running.

## Understanding Response Engine and Security Playbooks {#understandingresponseengineandsecurityplaybooks}

There are a few parts that compose a security runtime framework:

*   A **runtime visibility agent** that can provide live visibility and observability of the container internal behavior (open files, open network connections, processes executed, etc).
    *   Behavioral rules that can determine what is considered "normal" for this container and what should be notified.
    *   Indicators of compromise that can identify possible anomalous activity, a symptom of a potential security intrusion.
*   A **response engine**, reacting to the events of the monitoring agent, from just routing the warning messages to a notification endpoint or a logging or audit system, to performing more preventive actions to stop or mitigate the attack.
*   **Security playbooks** contain the steps to be taken to respond to a security incident. These can be either a manual process or written as code and run in an automated way.

While detecting anomalous behavior in your containers is step one, and step two is responding to or mitigating the attack in an automated way. In the Docker and Kubernetes world, it's important to keep in mind that:

*   Kubernetes clusters span a **large number of nodes and container deployments**, which can make it challenging to correlate the runtime alert and the originating container.
*   Any reaction to a security incident **needs to be fast**. Because containers have short lifespans, it's essential to find the culprit before the container causes any real damage or simply disappears or moves somewhere else. Automation is more important than ever, and writing response procedures as code playbooks is the best approach.

## Runtime Container Security Open Source Tools {#containerruntimesecurityopensourcetools}

There are a variety of Linux runtime security open source tools. Seccomp, SELinux/Auditd and Appamor are traditional system call audit and enforcement tools that can be applied to container.

<a href="https://sysdigrp2rs.wpengine.com/opensource/falco/" target="_blank">Sysdig Falco</a> focuses on behavior monitoring. In a nutshell:

*   Falco gains **visibility inside the containers** through system call instrumentation, with great performance and very little overhead, so can run in production systems.
*   System call **instrumentation is completely transparent** to your containers, so you don't need to modify your code, container images or inject or preload libraries. System call interception is done either through a simple kernel module dynamically compilled with DKMS or through an eBPF probe.
*   When an anomalous activity is detected, a security event, like an alert is emitted. The conditions that trigger the alert are defined by your policy, a collection of rules which syntax is easy, simple and very similar to tcpdump and Sysdig. You will find yourself writing rules before you realize.
*   Falco is **container native**, so rules and alerts are going to understand what is a process but also a container or a Kubernetes pod.

To learn more about how other runtime security tools compare to Falco, have a look at "<a href="https://sysdigrp2rs.wpengine.com/blog/selinux-seccomp-falco-technical-discussion/" target="_blank">SELinux, Seccomp, Sysdig Falco, and you: A technical discussion</a>".

Falco is extremely effective in part because its rules can use **context from the low level operating system interface (syscalls) together with Docker container or Kubernetes orchestration metadata** (container information, namespaces, deployment, pods, etc).

For example, you can build a Falco rule that will detect any listening socket outside our expected one, when:

*   The container image is `myregistry/nginx`.
*   The listening process inside that container is `nginx`.
*   The Kubernetes namespace is `load-balancer`.

Anything else should trigger an alert:

    condition: evt.type in (accept,listen) and (container.image!=myregistry/nginx or proc.name!=nginx or k8s.ns.name!="load-balancer")

This is a good example combining conditions from different sources:

*   System call events:
    *   i.e. evt.type = listen, evt.type = mkdir, evt.type = setns, etc.
*   Docker metadata:
    *   i.e. container.image, container.privileged, container.name, etc.
*   Process tree information:
    *   i.e. proc.pname, proc.cmdline, etc.
*   Kubernetes namespace metadata:
    *   i.e. k8s.ns.name, k8s.pod.name, etc.

It provides the flexibility and expressiveness needed to create accurate security rules that fully understand your operational entities.

## Building a Kubernetes Runtime Security Stack with Open Source Tools {#buildingakubernetesruntimesecuritystackwithopensourcetools}

While Falco provides deep visibility into what's happening inside containers, creating a complete container security stack requires additional components. There are open source components that can be tied together to build a Kubernetes runtime security stack:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/kubernetes_runtime_security_stack-1.png" alt="Kubernetes Runtime Security Stack with Open Source Tools" width="960" height="556" class="alignleft size-full wp-image-7728" />][15] 

*   **Use case: Gain visibility inside your containers and applications running in Kubernetes, including cluster services.**
*   **Tool: <a href="https://github.com/draios/falco" target="_blank">Sysdig Falco</a>.** Falco provides great visibility and has Docker and Kubernetes native support. Install Falco in Kubernetes is really easy: it's shipped as a Docker container (amongst other options like deb/rpm) deployed with a DaemonSet, but we have also created a **Helm chart** to streamline configuration and deployment process.
  
*   **Use case: Define your runtime security policy for applications, services and the cluster.**
*   **Tool: <a href="https://github.com/draios/falco-extras" target="_blank">Falco default ruleset library for the most popular Docker images</a>.** Creating Falco rules isn't difficult but can be annoying for those new to runtime security. Leverage our **library of default rulesets for the most popular Docker images** including kube-system components, Nginx, HAproxy, Apache, Redis, MongoDB, Elastic, PostgreSQL, etc.
  
*   **Use case: Response engine to forward security events based on your policy violations.**
*   **Tool:** **<a href="https://nats.io/" target="_blank">NATS</a>** Falco evaluates the rules and triggers alerts. A small forwarder in a container within the same pod reads the alerts through a named pipe and then sends them to NATS using TLS. NATS is a **messaging broker** so other parties can consume the security events.
  
*   **Use case: Incident response through security playbooks.**
*   **Tool: <a href="https://kubeless.io/" target="_blank">Kubeless</a>** for FaaS scripts. Kubeless is a Function as a Service framework for Kubernetes. Will subscribe and listen for events on NATS executing different **playbooks for incident response and attack mitigation, automated actions written as code**. For example, will post a notification to Slack, connect to the Kubernetes API server to stop the pod, or create a Network Policy that will isolate the pod from the network.
  
*   **Use case: Logging, audit and reporting.**
*   **Tool: EFK.** Falco doesn't implement any persistent storage. If you need to check the historical event log, perform data analysis, reporting etc. you can easily integrate Falco with third party logging tools like EFK: Fluentd + Elastic + Kibana.

  
## Gain Visibility Inside your Containers and Applications with Sysdig Falco (installation steps included) {#gainvisibilityinsideyourcontainersandapplicationswithsysdigfalcoinstallationstepsincluded}

If you are already using <a href="https://github.com/kubernetes/helm" target="_blank">Helm</a> for managing your Kubernetes applications, then you can install Falco in a few seconds with a simple command using the <a href="https://github.com/kubernetes/charts/tree/master/stable/falco" target="_blank">Falco Helm chart</a>:

    $ helm install --name sysdig-falco-1 --set integrations.natsOutput.enabled=true stable/falco

This method provides several advantages:

*   All the benefits of installing Falco as a Kubernetes DaemonSet, so you don't need to repeat the deployment on new nodes that will get Falco automatically.
*   Easier, faster, automated.
*   Configuration and rulesets managed by the chart, creating portable and repeatable configuration.
*   Bundles some integrations out of the box, like Kubernetes RBAC permissions.

You have more details about this installation method on "<a href="https://sysdigrp2rs.wpengine.com/blog/falco-helm-chart/" target="_blank">Automate Sysdig Falco Deployment Using Helm Charts</a>".

## Define your Runtime Security Policy for Applications, Services and the Cluster with Falco Default Ruleset Library {#defineyourruntimesecuritypolicyforapplicationsservicesandtheclusterwithfalcodefaultrulesetlibrary}

Most of us build our Cloud Native applications using open-source components. Thus, we have created a library with a growing list of Falco default ruleset for the most popular container images.

If you are using these images to build for services or as a base image to containerize your applications, you can simply adapt volume paths and maybe network connections to start. In the<a href="https://github.com/draios/falco-extras" target="_blank"> GitHub repository: falco-extras</a>, you will find rules for:

*   Kubernetes 1.10 cluster components:
    *   apiserver
    *   controller-manager
    *   kube-dns
    *   kube-scheduler
    *   dashboard
*   Google Kubernetes Engine components
*   Apache
*   Consul
*   ElasticSearch
*   etcd
*   Fluentd
*   HAproxy
*   MongoDB
*   Nginx
*   PHP-FPM
*   PostgreSQL
*   Redis
*   Traefik

As example, let's show how the Nginx ruleset would detect an `ls` command:

    # cp ~/falco-extras/rules/rules-nginx.yaml /etc/falco/falco_rules.local.yaml
    # service falco restart
    $ docker run -d -P --name mynginx nginx
    $ docker exec mynginx ls
    $ cat /var/log/falco.log
    11:03:13.464648222: Notice Unexpected process spawned in nginx container (command=ls  pid=26942 user=root mynginx (id=4f94bdd87187) image=nginx)

The `ls` command is not a whitelisted binary in <a href="https://github.com/draios/falco-extras/blob/master/rules/rules-nginx.yaml" target="_blank">this template</a>.

Using these default Falco runtime security rulesets saves time otherwise spent writing runtime security policy for common images. However, keep in mind that every version or even tag of a Docker container image is unique, and may have differences in user-defined data directories, binary paths, scripts that need to access some external port or device, or the configuration. You will need to adapt the templates to your specifics before actually using them in production.

If you want to use these rulesets from the library plus you own customization with the Helm chart, use this script to generate the Helm configuration:

    $ git clone https://github.com/draios/falco-extras.git
    $ cd falco-extras
    $ ./scripts/rules2helm rules/rules-traefik.yaml rules/rules-redis.yaml > custom-rules.yaml
    $ helm install --name sysdig-falco-1 -f custom-rules.yaml stable/falco

There are complete instructions, practical examples and more information about the library of Falco default rulesets in the "<a href="https://sysdigrp2rs.wpengine.com/blog/docker-runtime-security/" target="_blank">Implementing Docker/Kubernetes runtime security</a>".

## Kubernetes Incident Response {#kubernetesincidentresponse}

### Deploying Kubernetes Response Engine Components: NATS and Kubeless framework {#deployingkubernetesresponseenginecomponentsnatsandkubelessframework}

In order to install the NATS and Kubeless components in your Kubernetes cluster, you need to meet the following prerequisites:

*   The `kubectl` command and configured to access your Kubernetes cluster.
*   The `pipenv` python package. If you haven't installed it yet, you can use `pip`:

    $ pip install --user pipenv

*   The `kubeless` command. Have a look at the <a href="https://kubeless.io/docs/quick-start/" target="_blank">Kubeless Quick Start guide</a> to learn how to install it.

Once you have checked the list above, you just need to clone the Falco repo and then:

    $ git clone https://github.com/draios/falco.git
    $ cd integrations/kubernetes-response-engine
    $ cd deployment
    $ make

The make will use `kubectl` to deploy NATS using a Kubernetes Operator and the Kubeless framework that makes use of Kubernetes Custom Resource. The `nats-operator` might require additional time to complete the deployment and begin processing objects `kind: NatsCluster`. Once the deployment is finished, you can configure the different Kubernetes security playbooks.

### Deploying Kubernetes Security Playbooks on Kubeless {#deployingkubernetessecurityplaybooksonkubeless}

The response engine will classify and store the security events, providing a reliable timestamping, publishing, queuing and subscription mechanism. Now we need a consumer that is subscribed to an event or set of events and triggers the desired action, response or mitigation, this is what we call a "Kubernetes security playbook".

Our playbooks are currently implemented using Kubeless functions. Kubeless allows us to deploy a function and subscribe to a NATS topic, so it's a perfect match.

Every time we receive a matching NATS message, our Kubeless function will be run. This design allows for modularity and composability, you can run independent functions that react to each topic filter or an entire category.

The sequence of a Falco alert until a playbook function is triggered will follow a journey through the different components of the system:

1.  Falco evaluates the rules in real time and writes any triggered alert into a Linux pipe so it doesn't require any persistent storage in that Kubernetes pod.

2.  A sidecar container `falco-nats` receives these events through the pipe and performs some basic formatting to facilitate metadata extraction. Then queues the notification on NATS, published under a topic with the following format:

`falco.{severity}.{rule name slugified}` i.e. `falco.error.write_below_etc` 

<ol start="3">
  <li>
    The Kubeless functions can subscribe to the different NATS topics, which allows you to group, filter or ignore the different alerts as you see fit, performing a different action per group/filter you declare. For example, any message on <code>falco.warning.*</code> will trigger a notification, but <code>falco.error.write_below_etc</code> in particular will trigger a container deletion.
  </li>
</ol>

#### **Kubernetes Security Playbook: Slack Webhook Notification** {#kubernetessecurityplaybookslackwebhooknotification}

Let's start with sending a notification using a webhook.

For example, you can create a <a href="https://api.slack.com/slack-apps" target="_blank">Slack app</a>, and once you have authorized and configured the details, you will be able to post messages to a channel using a custom URL.

You can easily create a "catch-all" notification function:

    $ cd reactions
    $ ./deploy_playbook -r slack -e SLACK_WEBHOOK_URL=https://<custom_slack_app_url> -t "falco.*.*"

This deploys the Kubeless function that subscribes to the NATS `falco.*.*` topic(s).

If you need to specify multiple options, you can use the `-e` parameter several times. You can also subscribe a Kubeless function to several topics using the `-t` parameter multiple times.

If you just want to test things, Falco includes a `falco-event-generator` pod that intentionally fires several Falco alerting conditions included in the default configuration. You should start receiving these events in your Slack channel within a few seconds:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/falco_event_runtime_response_engine.png" alt="Falco Playbook Runtime Response Engine" width="1211" height="848" class="alignleft size-full wp-image-7730" />][16] 

#### **Kubernetes Security Playbook: Delete Offending Pod** {#kubernetessecurityplaybookdeleteoffendingpod}

A more strict reaction to respond an incident could be directly killing the Pod that triggered the security alert. This is done connecting to the Kubernetes API and deleting that Pod.

You can also deploy this function to your Kubeless using a similar command:

    $ ./deploy_playbook -r delete -t "falco.notice.terminal_shell_in_container"

Everytime we receive a `Terminal shell in container` alert from Falco this reaction will kill the container stopping the Pod via the Kubernetes API.

#### **Kubernetes Security Playbook: Taint a Node and Stop Scheduling** {#kubernetessecurityplaybooktaintanodeandstopscheduling}

A very interesting response would be to put aside the node where the offending container was running, so nothing else is scheduled there. You can assign flags (<a href="https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/" target="_blank">taints and tolerations</a>) to the Kubernetes node where the alert originated to achieve different effects.

    $ ./deploy_playbook -r taint -t "falco.notice.contact_k8s_api_server_from_container" [-e PARAMETER]

Parameters:

*   `TAINT_KEY`: This is the taint key. Default value: `falco/alert`
*   `TAINT_VALUE`: This is the taint value. Default value: `true`
*   `TAINT_EFFECT`: This is the taint effect. Default value: `NoSchedule`

In the example, we are executing without parameters, it will use the default taint `falco/alert=true:NoSchedule` so no news Pods will be scheduled in this node unless they can tolerate that specific taint. You can use a more aggressive approach and set `-e TAINT_EFFECT=NoExecute`, which will drain the affected node.

#### **Kubernetes Security Playbook: Pod Network Isolation** {#kubernetessecurityplaybookpodnetworkisolation}

If you suspect the attacker is trying to hijack its neighbors and spread across the cluster (doing network scanning, vulnerability probing, connecting to external IP addresses, etc), then you can isolate the Pod from the network.

The "isolate" reaction denies all ingress/egress traffic from a Pod. This reaction requires a Kubernetes cluster / Pod overlay network that supports <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-psp-network-policy/#kubernetesnetworkpolicies" target="_blank">Kubernetes Network Policies</a>.

    $ ./deploy_playbook -r isolate -t "falco.notice.unexpected_network_connection"

So as soon as we notice an unexpected network connection, we <a href="https://kubernetes.io/docs/concepts/services-networking/network-policies/#isolated-and-non-isolated-pods" target="_blank">isolate</a> that Pod, avoiding further network activity.

## Kubernetes Security Logging with Falco & Fluentd {#kubernetessecurityloggingwithfalcofluentd}

In addition to running playbooks, you may want to have a long-term event storage to enable reporting, historical trend visualization or data mining of your security events.

Fluentd is deployed by default on several Kubernetes distributions so we decided to go with the <a href="https://docs.fluentd.org/v0.12/articles/kubernetes-fluentd" target="_blank">EFK</a> (Fluentd, Elastic, Kibana) stack here.

Enable the Falco JSON-formatted event output to stdout and Fluentd will take it from there. You can find the complete deployment instructions and configuration options in "<a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-logging-fluentd-falco/" target="_blank">Kubernetes Security Logging with Falco & Fluentd</a>" to learn how to:

*   Deploy the EFK stack in your Kubernetes cluster
*   Deploy a compatible Falco DaemonSet that gets scraped by Fluentd
*   Filter the relevant Falco events from the bulk input
*   Configure awesome event visualizations in Kibana

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/kubernetes_efk_stack_response_engine_playbook.png" alt="Kubernetes EFK Stack Response Engine Playbook" width="1334" height="688" class="alignleft size-full wp-image-7731" />][17] 

Falco security events in Kibana

But if you are already using a SIEM product, then consolidating Falco alerts there is also quite easy. As an example we wrote a <a href="https://sysdigrp2rs.wpengine.com/blog/falco-gke-kubernetes-security/" target="_blank">Falco connector for Google Cloud Security Command Center</a>.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/Kubernetes_response_engine_playbook_google_scc.png" alt="Kubernetes Response Engine Playbook Google Security Command Center" width="1000" height="484" class="alignleft size-full wp-image-7732" />][18] 

Falco security events with Kubernetes information in SCC

## Conclusions {#conclusions}

Runtime container security is crucial to Docker and Kubernetes security strategy, and can be implemented using open source tools.

In this post we have explained how to integrate the Falco runtime security tool with some of the typical Cloud Native Kubernetes tooling like Helm, NATS and Kubeless to help you build a comprehensive open source container security stack.

Excited already? We hope you can use this as a whole open source container security stack or you might just pick the ideas and tools that are more useful to you, customize and adapt them to your use cases.

But we haven't finished yet, in addition to runtime security, we are also going to integrate Docker image scanning, continue reading into How to Implement Open Source Container Security: Part 2 <a href="https://sysdigrp2rs.wpengine.com/blog/container-security-docker-image-scanning/" target="_blank">Docker Image Scanning</a>.

If you liked this and you want to contribute to this project, there are many ways to do so:

*   Help us to improve this guide and create more documentation.
*   Extend the <a href="https://sysdigrp2rs.wpengine.com/blog/docker-runtime-security/" target="_blank">library of default Falco rules</a>.
*   Contribute more <a href="https://github.com/draios/falco/tree/dev/integrations/kubernetes-response-engine" target="_blank">security playbooks as Kubeless FaaS</a> or any other NATS observers.
*   Or integrate more tools, PRs are always great!

Join the discussion on Sysdig's open source <a href="https://sysdig.slack.com" target="_blank">Slack community</a>, #open-source-sysdig or reach out us via Twitter on <a href="https://twitter.com/sysdig" target="_blank">@sysdig</a>!

 [1]: #runtimesecurity
 [2]: #understandingresponseengineandsecurityplaybooks
 [3]: #containerruntimesecurityopensourcetools
 [4]: #buildingakubernetesruntimesecuritystackwithopensourcetools
 [5]: #gainvisibilityinsideyourcontainersandapplicationswithsysdigfalcoinstallationstepsincluded
 [6]: #defineyourruntimesecuritypolicyforapplicationsservicesandtheclusterwithfalcodefaultrulesetlibrary
 [7]: #kubernetesincidentresponse
 [8]: #deployingkubernetesresponseenginecomponentsnatsandkubelessframework
 [9]: #deployingkubernetessecurityplaybooksonkubeless
 [10]: #kubernetessecurityplaybookslackwebhooknotification
 [11]: #kubernetessecurityplaybookdeleteoffendingpod
 [12]: #kubernetessecurityplaybooktaintanodeandstopscheduling
 [13]: #kubernetessecurityplaybookpodnetworkisolation
 [14]: #kubernetessecurityloggingwithfalcofluentd
 [15]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/kubernetes_runtime_security_stack-1.png
 [16]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/falco_event_runtime_response_engine.png
 [17]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/kubernetes_efk_stack_response_engine_playbook.png
 [18]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/07/Kubernetes_response_engine_playbook_google_scc.png