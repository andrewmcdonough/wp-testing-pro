---
ID: 6300
post_title: Sysdig Secure February Release
author: Knox Anderson
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-secure-february-release/
published: true
post_date: 2018-02-22 14:10:07
---
We’ve been busy at work over the winter adding new functionality to Sysdig Secure and wanted to round up some of the features we released over the holiday season and in first month of the year. There are three main themes in this February release.

*   [Kubernetes Oriented Security- Security event topologies, default dashboards, & policy management][1]

*   [Improved Policy Management - New default policies & updated falco rule editor][2]

*   [New Enterprise Integrations - Secure authentication & notification channels][3]

### Kubernetes Oriented Security {#kubernetesorientedsecurity}

Many of the new policies and pieces of functionality we introduced revolved around tighter integrations with the orchestrator. When customers get their first containerized application into production they often think about what is needed to protect the container, when their real goal is delivering a stable, secure service. To make it easier to view which policies are protecting hosts, containers, services, cloud regions, etc we’ve add the ability to group policies by scope.

[<img style="max-width: 100%; height: auto;" src="/wp-content/uploads/2018/02/Policy-events-grouped-by-scope.png" alt="Policy Events Grouped by Scope" width="1200" height="750" class="alignleft wp-image-4350" />][4]

*This shows how policies can be viewed based on the scope to which they apply. Here we can see the specific policies configured to protect containers in my prod kubernetes namespace.*

**Topology Maps**

We also released Topology maps for event analysis. Topology maps allow Security Analysts to quickly identify where events have happened on a host or service, and escalate to the proper development and operations teams in charge of those services. The maps are also useful to quickly identify any network dependencies between hosts, containers, and services as well as uncover unexpected connections. Depending on the scope, analysts can see all the entities in a grouping hierarchy, the network connections between them and the count of events that have happened during the timeframe. In the example below we show what topology maps look like from both a physical and logical perspective.

[<img style="max-width: 100%; height: auto;" src="/wp-content/uploads/2018/02/topologies-secure-e1519249130502.gif" alt="Secure Topologies" width="1200" height="750" class="alignleft wp-image-4350" />][5]

*This map first shows the events in kubernetes cluster from a physical perspective, and the a logical view based on the Kubernetes metadata. Notice how much easier it is to detect what applications have events in the service-oriented view.*

**Overview Dashboard**

Policy events can also be visualized through the overview dashboard within the events page. The overview dashboard gives analysts and administrators a quick overview of which policies, hosts, containers, and deployments had the most events over the selected timeframe. It’s also a great way to get an at a glance summary of how many hosts the agent is installed on and the severity of the events that have occurred.

[<img style="max-width: 100%; height: auto;" src="/wp-content/uploads/2018/02/overview-dashboard-secure-e1519249081741.gif" alt="Secure Overview Dashboard" width="1200" height="750" class="alignleft wp-image-4350" />][6]

*The overview dashboard is a good way to bridge the gap between your SecOps team and developers. Quickly identify images and applications that have had incidents to set priority with developers. *

### New Kubernetes & AWS Policies {#newkubernetesawspolicies}

We also added two new default policies to cover Kubernetes & AWS API access. The Kubernetes API has been a popular target for <a href="https://sysdigrp2rs.wpengine.com/blog/detecting-cryptojacking/" target="_blank">cryptojacking</a> so we added a new policy to detect any connection to the K8s API Server besides those that are explicitly allowed.

<pre>- rule: Contact K8S API Server From Container
  desc: Detect attempts to contact the K8S API Server from a container
  condition: outbound and k8s<em>api</em>server and container and not k8s_containers
  output: Unexpected connection to K8s API Server from container (command=%proc.cmdline %container.info image=%container.image connection=%fd.name)
  priority: NOTICE
  tags: [network, k8s, container]</pre>

We also added another policy around API services where we’ll detect unexpected attempts from containers to communicate with the EC2 Instance Metadata Service.

<pre>- rule: Contact EC2 Instance Metadata Service From Container
  desc: Detect attempts to contact the EC2 Instance Metadata Service from a container
  condition: outbound and fd.sip="169.254.169.254" and container and not ec2<em>metadata</em>containers
  output: Outbound connection to EC2 instance metadata service (command=%proc.cmdline connection=%fd.name %container.info image=%container.image)
  priority: NOTICE
  tags: [network, aws, container]</pre>

### New Policy Editor {#newpolicyeditor}

We’ve made it easier for users to bring their existing falco rules and add new rules to Sysdig Secure by adding a rules editor in the Sysdig Secure interface. Just copy in any custom rule, save it, and it will be added to the default ruleset and available within the policy editor.

[<img style="max-width: 100%; height: auto;" src="/wp-content/uploads/2018/02/Rules-Editor.png" alt="Falco Rule Editor" width="1200" height="750" class="alignleft wp-image-4350" />][7]

*Quickly add a new rule to your default falco rule set directly within the UI. *

### Enterprise Integrations - SSO authentication & new notification channels {#enterpriseintegrationsssoauthenticationnewnotificationchannels}

**Single Sign On Authentication** 
We wanted to streamline your user experience by releasing Single Sign On (SSO) for Sysdig Secure.

[<img style="max-width: 100%; height: auto;" src="/wp-content/uploads/2018/02/SSOEtc-Login.png" alt="SSO" width="1200" height="750" class="alignleft wp-image-4350" />][8]

Our SSO solution provides support for the most common methods of SSO, across both our cloud offering and our on-premise software offering:

*   Google Authentication

*   SAML

*   OpenID (Cloud)

*   LDAP (On-premise)

Find more details on how to configure <a href="https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/115003573843-SAML" target="_blank">SAML</a> and <a href="https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/115003889603" target="_blank">OpenID</a>.

**New Notification Channels**

We’ve also added support for new notification channels as well as webhooks to route any event to a Incident Management ticketing system, or some your SIEM of choosing.

[<img style="max-width: 100%; height: auto;" src="/wp-content/uploads/2018/02/Notification-Channels.png" alt="Notification Channels" width="1200" height="750" class="alignleft wp-image-4350" />][9]

 [1]: #bookmark=id.gpt1w6ty9q52
 [2]: #bookmark=id.kpy3w3ndjjr8
 [3]: #bookmark=id.utmum0mhlx88
 [4]: /wp-content/uploads/2018/02/Policy-events-grouped-by-scope.png
 [5]: /wp-content/uploads/2018/02/topologies-secure-e1519249130502.gif
 [6]: /wp-content/uploads/2018/02/overview-dashboard-secure-e1519249081741.gif
 [7]: /wp-content/uploads/2018/02/Rules-Editor.png
 [8]: /wp-content/uploads/2018/02/SSOEtc-Login.png
 [9]: /wp-content/uploads/2018/02/Notification-Channels.png