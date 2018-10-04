---
ID: 9949
post_title: >
  Docker scanning for Jenkins CI/CD
  security with the Sysdig Secure plugin.
author: Knox Anderson
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/docker-scanning-jenkins/
published: true
post_date: 2018-09-05 01:14:01
---
<span style="font-weight: 400;">In this blog post we’ll cover how to implement Docker Scanning for Jenkins with the <a href="https://wiki.jenkins.io/display/JENKINS/Sysdig+Secure+Jenkins+Plugin">Sysdig Secure Jenkins plugin</a>. The plugin can be used in both freestyle and pipeline jobs to scan images and fail the build if the image fails a policy evaluation.</span>

<span style="font-weight: 400;">The deployment model of containers has made it incredibly easy for organizations to adopt continuous delivery processes. However, all the efficiencies gained in packaging and building applications can’t be realized if the end result is unstable and insecure software. By prioritizing <a href="https://sysdigrp2rs.wpengine.com/use-cases/continuous-security/">CI/CD security</a> organizations can proactively address risk in applications before they are deployed in production, or even pushed into a registry. </span>

### <span style="font-weight: 400;">Fail Fast: The Benefits of CI/CD Security</span>

<span style="font-weight: 400;">It’s always easier to fix issues when they’re not in production. By integrating Sysdig Secure with your CI/CD pipeline with Jenkins or any other tool a step is added to evaluate Docker images for security, compliance, and reliability before deploying images to production.</span>

<span style="font-weight: 400;">Here are a couple examples of things we’ve seen organizations want to know about images before they’re deployed into production.</span>

<span style="font-weight: 400;">Security </span>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">Does the image have critical vulnerabilities with a fix?</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Are there secrets or credentials exposed in the image?</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Does this image have exposed ports that I’ve blacklisted?</span>
</li>

<span style="font-weight: 400;">Compliance</span>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">What license types is the image using?</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Is this image built on an distribution our organization doesn’t use?</span>
</li>

<span style="font-weight: 400;">Reliability</span>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">Does my image have health checks?</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Are my developers building large images that can impact our infrastructure?</span>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Are my developers using an unofficial version of Ruby, Node, Java, or Python packages?</span>
</li>

### <span style="font-weight: 400;">Scanning Docker Images built with  Jenkins</span>

<span style="font-weight: 400;">There’s a couple prerequisites to cover before scanning Docker images built within Jenkins. </span>

<li style="font-weight: 400;">
  <span style="font-weight: 400;">Have a SaaS or On-prem installation of </span><a href="https://sysdigrp2rs.wpengine.com/products/secure/"><span style="font-weight: 400;">Sysdig Secure</span></a>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Install the </span><a href="https://wiki.jenkins.io/display/JENKINS/Sysdig+Secure+Jenkins+Plugin"><span style="font-weight: 400;">Sysdig Secure Jenkins Plugin</span></a>
</li>
<li style="font-weight: 400;">
  <span style="font-weight: 400;">Configure the plugin to integrate with Sysdig Secure (shown below)</span>
</li>

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/ci-cd-security-1-1170x243.png" alt="" width="640" height="133" class="alignnone wp-image-9957 size-large" />][1]  
### <span style="font-weight: 400;">Creating Docker Image Scanning Policies for Jenkins in Sysdig Secure</span>

<span style="font-weight: 400;">Once Sysdig Secure and Jenkins are integrated, it’s time to set up a policy to be used by the Jenkins plugin. </span>**Note:** **This is not required and the plugin will use the default policy within Sysdig Secure if a custom policy is not configured.**

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/ci-cd-security-2.gif" alt="" width="1425" height="732" class="alignnone wp-image-9956 size-full" />][2]

<span style="font-weight: 400;">Navigate to the</span>*<span style="font-weight: 400;"> Scanning Policies</span>*<span style="font-weight: 400;"> page within Sysdig Secure and click on </span>*<span style="font-weight: 400;">Add Policy</span>*<span style="font-weight: 400;"> to get started. You can easily configure rules to map to the security, compliance, and reliability uses cases we provided above plus many more.<br /></span>

<span style="font-weight: 400;">The last step of creating a rule is to assign an action of </span>*<span style="font-weight: 400;">Warn</span>* <span style="font-weight: 400;">or </span>*<span style="font-weight: 400;">Stop</span>***. **<span style="font-weight: 400;">The </span>*<span style="font-weight: 400;">Stop</span>*<span style="font-weight: 400;"> action can be used to fail a build and prevent the image from moving into production. </span>

### <span style="font-weight: 400;">Scanning Docker Images as part of the CI/CD Pipeline with Jenkins</span>

<span style="font-weight: 400;">Once you’ve set up a policy it’s time to integrate that policy evaluation into an existing build process within Jenkins. Full documentation can be seen in the </span>[<span style="font-weight: 400;">Sysdig Secure Jenkins Plugin documentation</span>][3]<span style="font-weight: 400;">.</span>

<span style="font-weight: 400;">First, add the additional build step </span>*<span style="font-weight: 400;">Sysdig Secure Container Image Scanner</span>***:  
[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/ci-cd-security-4-350x260.png" alt="" width="350" height="260" class="alignnone wp-image-9954 size-medium" />][4]  
**

<span style="font-weight: 400;">Then you’ll have options to define which policy you’d like this job to reference and whether or not to fail a build based on a policy failed policy evaluation (if there are any stop actions).<br /><a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/ci-cd-security-3.png"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/ci-cd-security-3-1170x471.png" alt="" width="640" height="258" class="alignnone wp-image-9955 size-large" /></a><br /></span>

### <span style="font-weight: 400;">Reporting on Docker Image Risk and Compliance within Jenkins</span>

<span style="font-weight: 400;">After the next build an additional Sysdig Secure Report artifact will now be available in Jenkins.<br /><a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/ci-cd-security-5.png"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/ci-cd-security-5.png" alt="" width="1928" height="1150" class="alignnone wp-image-9953 size-full" /></a><br /></span>

<span style="font-weight: 400;">By clicking into the </span>*<span style="font-weight: 400;">Sysdig Secure Report</span>*<span style="font-weight: 400;"> you’ll get an summary of the policy evaluation broken down by the different stop or warn actions that were generated from the policy.<br /><a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/ci-cd-security-6.gif"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/ci-cd-security-6.gif" alt="" width="1425" height="732" class="alignnone wp-image-9952 size-full" /></a><br /></span>

<span style="font-weight: 400;">To dive further into a report about the specific vulnerabilities of an image click on the </span>*<span style="font-weight: 400;">Security</span>*<span style="font-weight: 400;"> tab and a page specific to vulnerabilities will open.<br /><a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/ci-cd-security-7.png"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/ci-cd-security-7.png" alt="" width="2202" height="1242" class="alignnone wp-image-9958 size-full" /></a><br /></span>

### <span style="font-weight: 400;">Tying it all together</span>

<span style="font-weight: 400;">All this data is also sent to the Sysdig Secure UI where you can get further details about the image, OS package information, configuration files, discovered vulnerabilities and any possible leaked secrets or credentials, and a view into if & where this image is currently running in your environment.<br /><a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/ci-cd-security-8.gif"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/ci-cd-security-8.gif" alt="CI/CD Security" width="1425" height="732" class="alignnone wp-image-9959 size-full" /></a><br /></span>

<span style="font-weight: 400;">Also it’s worth noting that everything you see here can also be accomplished directly via the API. So if you’re using other CI/CD tools besides Jenkins we’ll easily integrate with those as well.</span>

<span style="font-weight: 400;">If you’d like to learn more about how Sysdig Secure can integrate with your CI/CD to help manage risk, compliance, and reliability check out this </span>[<span style="font-weight: 400;">How to manage vulnerabilities in container environments online session</span>][5]<span style="font-weight: 400;">.</span>

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/ci-cd-security-1.png
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/ci-cd-security-2.gif
 [3]: https://wiki.jenkins.io/display/JENKINS/Sysdig+Secure+Jenkins+Plugin
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/ci-cd-security-4.png
 [5]: https://www.brighttalk.com/mybrighttalk/channel/16287/webcast/331984/brighttalkhd