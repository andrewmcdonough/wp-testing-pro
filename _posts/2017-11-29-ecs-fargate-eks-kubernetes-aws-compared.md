---
ID: 5472
post_title: >
  ECS, Fargate and EKS (Kubernetes on AWS)
  compared and explained in a nutshell
author: Jorge Salamero Sanz
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/ecs-fargate-eks-kubernetes-aws-compared/
published: true
post_date: 2017-11-29 16:11:44
---
<p <Amazon has just announced on AWS re:Invent two new services relevant on the container ecosystem: Fargate and EKS (Elastic Kubernetes Service). With the information we have at this time, let’s explain and compare them against running Kubernetes on AWS.</p> </p>
Google and Azure both provide managed Kubernetes clusters since a few months now. <a href="https://blog.docker.com/2017/10/docker-enterprise-edition-kubernetes/" rel="noopener" target="_blank">Docker announced Kubernetes support</a> in their commercial offering just a few weeks ago. Even <a href="https://mesosphere.com/blog/docker-vs-kubernetes-vs-apache-mesos/" rel="noopener" target="_blank">Mesosphere supports Kubernetes as a scheduler</a> over their DC/OS Mesos distribution. Like <a href="http://rancher.com/rancher-now-supports-kubernetes/" rel="noopener" target="_blank">Rancher did</a>.

Today Amazon bring us two related announcements:

*   <a href="https://aws.amazon.com/eks/" rel="noopener" target="_blank">EKS (Elastic Kubernetes Service)</a>, see the <a href="https://aws.amazon.com/blogs/aws/amazon-elastic-container-service-for-kubernetes/" rel="noopener" target="_blank">announcement here</a>
*   <a href="https://aws.amazon.com/blogs/aws/aws-fargate/" rel="noopener" target="_blank">Fargate (managed instances for ECS and EKS)</a>

## So, what’s AWS EKS?

Amazon Elastic Container Service for Kubernetes (Amazon EKS) is a managed Kubernetes service. It claims to use upstream Kubernetes and to be replicated across three masters in different Availability Zones. Uses IAM for RBAC, PrivateLink to reach the masters within your own private and VPC for pod networking.

## Managed ECS and Kubernetes with Fargate?

Fargate is an Amazon technology to run containers, either orchestrated by ECS or Kubernetes on their EKS (at some point in 2018), without having to manage the underlying EC2 instances.

ECS and EKS are just different schedulers, with different syntax, resources and capabilities to define how your containers are orchestrated. Fargate will execute and run these container, presumably using Docker. Will they support other containers via <a href="http://blog.kubernetes.io/2017/11/containerd-container-runtime-options-kubernetes.html" rel="noopener" target="_blank">containerd</a>?

Fargate tries to address two points:

*   People who don’t want to manage their ECS or Kubernetes instances
*   The <a href="https://en.wikipedia.org/wiki/Bin_packing_problem" rel="noopener" target="_blank">bin packaging problem</a>

When using Fargate you don’t pay for the instances but you pay per computing second used without having to worry about the underlying EC2 instances. And that pricing includes the managed service. How much extra is it? Well, it depends on multiple factors: density and packaging, scale and your infrastructure automation optimization. Let’s stick into the technical side of things in this discussion, we will let you do your own ToC calculation.

At the moment ECS has much better integration with other AWS services, and that was one of its selling points, but this might change as EKS progresses. EKS gives you the advantage of running the same scheduler in AWS or anywhere else, but this might change as EKS get more tightly integrated with other AWS services.

It seems that Fargate isolation will be at the cluster level, so like current ECS or Kubernetes deployments: containers running in the same cluster might share instances, but different clusters won’t.

Actually, if you spin up a Fargate ECS cluster, you won’t see the instances where it runs:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/ecs_fargate_managed_instances-1024x562.png" alt="" width="1024" height="562" class="aligncenter size-large wp-image-5475" />][1]

As we were used to with ECS traditionally:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/ecs_self-managed_instances-1024x562.png" alt="" width="1024" height="562" class="aligncenter size-large wp-image-5476" />][2]

## ECS or Kubernetes on AWS vs EKS with Fargate from a operations perspective

Fargate sounds like an interesting idea, and yes, it comes at a cost. But let’s play devil’s advocate role for a bit here. For those of us who already:

*   automated their infrastructure deployment in a declarative way with Cloudformation or Terraform (which are free)
*   use host minimalistic OS like CoreOS, Atomic or LinuxKit that highly simplify updates
*   who look at uptime as a metric that they want to reduce

Is such a pain maintaining your own container orchestration layer as long as you keep some provider independence? Or do you think biggest hassle is moving your applications into containers?

If the bin packaging problem is not already addressed for you with the move into containers, have you tried to play with Kubernetes requests, limits and <a href="http://blog.kubernetes.io/2017/03/advanced-scheduling-in-kubernetes.html" rel="noopener" target="_blank">advanced scheduling</a> features like affinity, taints, tolerations or <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-scheduler/" rel="noopener" target="_blank">custom schedulers</a>?

Kubernetes is a mix of core components plus a few of your own choice (SDN, service discovery / service mesh, ingress controller, persistent storage, logging, monitoring or tracing, etc). It is yet to be seen what kind of Kubernetes customization is allowed on EKS.

I would expect Amazon to provide native integration with all their services (VPC, ALB, EBS, CloudWatch or X-Ray), obviously at an extra cost, and this may be convenient for some, like the case of Kubernetes RBAC via IAM if you are already using a mix of AWS services plus Kubernetes.

But what if you want to customize or run your own components? Sounds unlikely possible, especially if going the Fargate managed way. We wait impatiently to play with this!

We are also curious about multi-zone replication and how that compares with Kubernetes federation? Will inter-zone traffic cost change compared with a self hosted Kubernetes design? I’ve already experienced the dilemma of running your own service discovery in ECS in order to avoid ELB/ALB costs for services that doesn’t need to be exposed outside.

[tweet_box design="default" float="none"] #ECS, Fargate and #EKS (#Kubernetes on #AWS) compared and explained in a nutshell #reInvent [/tweet_box] 

## ECS or Kubernetes vs Fargate for developers

From a developer perspective no having to manage your own cluster sounds like a very convenient idea. But if you were expecting to have quickly access to on-demand clusters, forget about it. Provisioning one is still quite slow, as it used to be with ECS:

*“We are creating resources for your service. this may take up to 10 minutes.”*

When spinning up some applications in to ECS + Fargate, I had the impression that the AWS console was just running some Cloudformation scripts under the hood, as I used to do until today:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/ecs_fargate_cloudformation-1024x341.png" alt="" width="1024" height="341" class="aligncenter size-large wp-image-5477" />][3]

Also it might be obvious for some, but remember that ECS tasks and service definition are completely different from Kubernetes pods, deployments and services definition. Either you go one or the other.

## And, Fargate compared to Lambda?

Farget is similar to Lambda as you pay per computing second used without having to worry about the EC2 instances. While if you select going the instances way you pay per computing resources reserved, no matter if you use them or not.

Lambda has its limitations on more complex applications like being stateless, not being able to handle warming up on start, or having a maximum execution time. All these can be implemented in your containerized application in ECS or Kubernetes.

## Conclusions

Finally Amazon is joining the Kubernetes club! There are a lot of unanswered questions around AWS Kubernetes service. Managed ECS via Fargate is available today. Does it worth the cost? Well, it depends on how mature is your infrastructure management and your workload. How will be with Fargate for EKS? We don’t know yet. As soon as we get access to EKS we will be updating this blog post with more information.

PD: Either if you are running Kubernetes in Google, Azure or AWS. GKE, AKS or ECS. Don’t miss Sysdig, our container native monitoring / run-time security and forensics tool. Managed SaaS or on-prem ;-) And signup into our blog for more updates on EKS!

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/ecs_fargate_managed_instances.png
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/ecs_self-managed_instances.png
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/ecs_fargate_cloudformation.png