---
ID: 9942
post_title: >
  Scanning images in Azure Container
  Registry.
author: Knox Anderson
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/scanning-images-in-azure-container-registry/
published: true
post_date: 2018-09-04 22:27:15
---
<span style="font-weight: 400;">With the 2.0 release of Sysdig Secure, we’re excited to support new integrations with services Azure provides around containers and Kubernetes. Today we’ll be diving deeper into how to integrate Sysdig Secure with ACR (Azure Container Registry) to scan images for</span><span style="font-weight: 400;"> for security, compliance, and reliability.</span><span style="font-weight: 400;"> Sysdig has offered </span>[<span style="font-weight: 400;">unified monitoring and security</span>][1]<span style="font-weight: 400;"> for container and Kubernetes deployments on Azure for years, and now with native CI/CD and registry integrations we’ve moved earlier into the developer lifecycle.</span>

### <span style="font-weight: 400;">About Azure Container Registry</span>

<span style="font-weight: 400;">The Azure Container Registry allows you to store container images for all types of orchestration platforms including Kubernetes, Docker or DC/OS and Azure services such as App Service, Batch, Service Fabric, and others. </span>

### <span style="font-weight: 400;">Azure Container Registry Security and Sysdig Secure</span>

<span style="font-weight: 400;">Scanning images in Azure Container Registry is the same as scanning from any other Docker v2 compatible registry. Once configured, the entire registry or individual images and tags can be analyzed and then evaluated against a Sysdig Secure Scanning policy.</span>

<span style="font-weight: 400;">The first step is to pass the ACR credentials into Sysdig Secure to give access to the registry. Once configured the Sysdig Secure scanning engine can pull any image stored within the registry into the engine for analysis. </span>

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/Azure-Container-Registry-Authentication-1170x597.png" alt="" width="640" height="327" class="alignnone wp-image-9944 size-large" />][2]  
<span style="font-weight: 400;">When an image is pulled into the scanning engine Sysdig Secure will provide visibility into: </span>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">Official OS packages</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Unofficial packages</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Configuration files</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Artifacts such as Javascript NPM modules, Python PiP, Ruby GEM, and Java JAR archives</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Secrets, credentials like tokens, certificates and other sensitive data</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Known vulnerabilities & available updates </span>
</li>

<span style="font-weight: 400;">These artifacts are then stored and evaluated against custom scanning policies to spot vulnerabilities, misconfiguration, or compliance issues within your images. </span>

### <span style="font-weight: 400;">Scanning Container Images in Azure Container Registry</span>

<span style="font-weight: 400;">Adding an image to the scanning engine from ACR is as simple as copying the registry URL/image/tag into the Sysdig Secure UI and clicking scan image. This process can also be easily scripted to import all images and to watch repositories for updates. </span>

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/scan-azure-image.gif" alt="" width="1431" height="708" class="alignnone wp-image-9945 size-full" />][3]  
<span style="font-weight: 400;">Once an image has been analyzed a report will be generated that has the outcome of the policy evaluation, all vulnerabilities discovered in OS packages, configuration files, and many other artifacts which are stored for audit and compliance reasons. </span>

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/azure-scanning-walkthrough.gif" alt="" width="1425" height="728" class="alignnone wp-image-9946 size-full" />][4]  
*<span style="font-weight: 400;">This report has details showing knoxsds.azurecr.io/cassandra:latest </span>**<span style="font-weight: 400;">has passed the policy evaluation, including image metadata, vulnerabilities, and even JAR archives information about what’s included in the image.</span>*

### <span style="font-weight: 400;">Conclusion</span>

<span style="font-weight: 400;">Hopefully you can see how easy it is to get up and running with both Azure Container Registry  and Sysdig Secure. If you’d like to see the integration in action sign up for a trial (both SaaS and On-prem options) or follow our blog for more post about securing containers on Azure. </span>

 [1]: https://azure.microsoft.com/en-us/blog/unifying-monitoring-and-security-for-kubernetes-on-azure-container-service/
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/Azure-Container-Registry-Authentication.png
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/scan-azure-image.gif
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/azure-scanning-walkthrough.gif