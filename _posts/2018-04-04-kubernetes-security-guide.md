---
ID: 6988
post_title: Kubernetes security guide.
author: Jorge Salamero Sanz
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-guide/
published: true
post_date: 2018-04-04 07:09:02
---
In this Kubernetes security guide we cover the most significant aspects of implementing Kubernetes security best practices.

Kubernetes security, like monitoring or building a CI/CD pipeline is becoming a must as a consequence of Kubernetes platform quickly gaining reputation as the *defacto* standard for modern containerized deployments.

Unfortunately, **traditional security processes need to be updated for Kubernetes**; legacy security tools that were not built for containers cannot look inside, analyze or protect containers, microservices and cloud-native apps. Containers are like black boxes, useful for moving applications from development into production. They provide a great level of portability and isolation. But makes harder to understand what's happening inside, monitor and secure them. Microservices help to develop applications faster and orchestration tools like Kubernetes allow to deploy and dynamically scale the apps, but now we have different pieces moving around, therefore we must take a more dynamic security approach.

Furthermore, many organizations are adopting security best practices and implementing **DevSecOps** processes, where everyone is responsible for security and, security is implemented from early development stages into production through the entire software supply chain, this also known as **Continuous Security**.

Moving Kubernetes to production implies new infrastructure layers, new components, new procedures and therefore new security processes and tools. This security guide will help you to implement Kubernetes security, mostly focused around Kubernetes specific security features and its configuration, but also some additionals tools that will go beyond what Kubernetes can do.

We have covered the following topics:

*   Chapter 1: <a target="_blank" href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-rbac-tls/">Understanding Kubernetes RBAC and TLS certificates</a>
*   Chapter 2: <a target="_blank" href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-psp-network-policy/">Implementing security at the pod level: Kubernetes Security Context, Kubernetes Security Policy and Kubernetes Network Policy</a>
*   Chapter 3: <a target="_blank" href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-kubelet-etcd/">Securing Kubernetes components (kubelet, etcd or your registry)</a>
*   Chapter 4: <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-harden-kube-system/" target="_blank">Hardening kube-system components</a> with Sysdig Secure security policies [tweet_box design="default" float="none"]#Kubernetes #security guide: how to implement RBAC, TLS, Pod Security Policies, Network Policies, etcd and kubelet security[/tweet_box] 

You can use this guide as comprehensive read if you are new to Kubernetes security or as a quick reference document / cheat sheet if you are looking at implementing specific Kubernetes security best practices.

If you are new into Docker and container security, as a prerequisite for this guide, we recommend you to check out <a href="https://sysdigrp2rs.wpengine.com/blog/7-docker-security-vulnerabilities/" target="_blank">7 Docker security vulnerabilities and threats</a>.

But if you are looking at expanding security into applications, you will find useful <a href="https://sysdigrp2rs.wpengine.com/blog/sysdig-secure-docker-run-time-security/" target="_blank">Kubernetes security policies and audit with Sysdig Secure</a> where we show how to secure Docker's example-voting-app, a modern microservices based demo app running on Kubernetes.

Additionally, we also wrote some real stories about Kubernetes security incidents that will help you understand the threats that you need to consider when running Kubernetes in production:

*   <a href="https://sysdigrp2rs.wpengine.com/blog/detecting-cryptojacking/" target="_blank">Fishing for Miners - Cryptojacking Honeypots in Kubernetes</a>.
*   <a href="https://sysdigrp2rs.wpengine.com/blog/detecting-cryptojacking-with-sysdigs-falco/" target="_blank">Detecting cryptojacking with Sysdig Falco</a> open-source.