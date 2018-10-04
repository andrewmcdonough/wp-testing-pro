---
ID: 7537
post_title: 'Sysdig Secure 2.0 &#8211; adds vulnerability management, 200+ compliance checks, and security analytics.'
author: Knox Anderson
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-secure-2-0/
published: true
post_date: 2018-06-14 00:36:19
---
A little over 2 years ago we opensourced <a href="https://sysdigrp2rs.wpengine.com/opensource/falco/" target="_blank">Sysdig Falco</a> with the goal of providing a robust detection engine that the community could use to securely run containers in production. Since the launch we expanded the default ruleset and have had 750,000+ downloads of Sysdig Falco. Organizations like <a href="https://cloud.gov/docs/apps/experimental/behavior-monitoring/" target="_blank">cloud.gov</a> and Yahoo have used Falco to detect behavioral anomalies across their containerized infrastructure.

We then integrated the falco engine into our [Cloud-native intelligence platform][1] and launched Sysdig Secure. Sysdig Secure 1.0 built on top of the deep visibility we had from Sysdig Monitor to provide run-time detection, audit, and forensics capabilities. Sysdig Monitor and Sysdig Secure share a unified agent that instruments the host OS and collects system calls. This single low impact agent provides monitoring, security, troubleshooting, and forensics capabilities without needing to instrument any individual containers or configure exporters. 

Today with the beta release of Sysdig Secure 2.0 we’ve extended the capabilities of our platform by adding vulnerability management, 200+ compliance checks, and security analytics. With native CI/CD and registry integrations as part of the Sysdig Cloud-Native intelligence platform we’ve moved our visibility earlier into the software development lifecycle. Let's take a look at some of the new features of the 2.0 release! 

## Vulnerability Management {#vulnerabilitymanagement}

Sysdig Secure’s vulnerability management capabilities help organizations bring application security, compliance, and quality closer to the developer. Through native integrations with common tooling in the software delivery chain Sysdig Secure enables teams to avoid and resolve security issues before a build is completed or a container is ever deployed. 

**Image Analysis** - Perform an inspection of an image to generate a detailed report of the contents of the image. Including:

*   Official OS Packages

*   Unofficial OS Packages 

*   Configuration Files

*   Language Modules - NPM, PiP, GEM, and Java Archives

*   Image metadata & more 

We’ll then automatically correlate the contents of the image with our vulnerability feeds to give insight into vulnerable packages, files, etc. 

**CI/CD Security** - Easily configure Sysdig Secure to scan images as part of your build process through a native Jenkins plugin or easily through API’s. Fail builds, trigger warnings, and enforce compliance easily by including image scanning every time a container goes through your build process. 

**Container Registry Integrations-** Scan images stored in any any Docker V2 compatible registry such as CoreOS Quay, Amazon ECR, Docker Private Registries, Google Container Registry or JFrog Artifactory, Microsoft ACR, SuSE Portus and VMWare Harbor.

**Policies -** Configure policies for your build pipeline or your registries to evaluate images against user defined policies for: vulnerabilities, operating system packages, 3rd party packages, software libraries, Dockerfile checks, file contents, configuration files, and image attributes. 

**Run-time Alerting -** Alert if unscanned images are deployed into production environments, if a new vulnerability is discovered in a package of an image that’s running in production, or if the scan status of one your running images changes. 

**Kubernetes Vulnerability Management** - tie back information about unscanned images, or scan results to Kubernetes clusters, namespaces, and deployments to categorize risk and prioritize image patching and upgrades. 

![Container Image Scanning][2] 
**Vulnerability Feeds -** Continuously updated vulnerability and package data from OS vendors, package repositories, and the National Vulnerability database.

## Compliance {#compliance}

Containers offer an opportunity for better security. They’re lightweight, have a smaller attack surface, and are immutable. However, the rate at which they’re deployed and [short lifecycle][3] means that configuration can be missed as time goes on.

Sysdig Secure 2.0 eases the pain of measuring and enforcing compliance across a distributed environment through compliance controls, audit checks, policies, and results. Sysdig automatically scans the infrastructure with requirements based on Center for Internet Security (CIS) configurations and hardening benchmarks. Users can scope and schedule Docker and Kubernetes benchmarks to measure and maintain compliance.

## Security Analytics {#securityanalytics}

One of the things every security team needs to be able to do is report on the status of their environment and how it’s changed over time. Sysdig Secure 2.0 provides rich metrics about events, compliance, and vulnerabilities enabling deeper analytics and a better understanding of the environment. Security metrics tied back to containers, images, hosts, and Kubernetes entities enable users to easily determine how different organizations, applications, and services are trending in risk and compliance.

![Dockerbench Dashboards][4] 
### What’s next? {#whatsnext}

If you’re interested in learning more about what Sysdig Secure can do, or want to see any of these features in action, [request a trial][5] or [demo][6]!

 [1]: https://sysdigrp2rs.wpengine.com/product/how-it-works/
 [2]: /wp-content/uploads/2018/06/Image-Scan-Runtime-Scan-In-Progress-Pie-Hover.png
 [3]: https://sysdigrp2rs.wpengine.com/blog/2018-docker-usage-report/
 [4]: /wp-content/uploads/2018/06/compliance_dashboards-1-e1528961092185.gif
 [5]: https://sysdigrp2rs.wpengine.com/product/secure/pricing/
 [6]: https://go.sysdigrp2rs.wpengine.com/docker-security-demo