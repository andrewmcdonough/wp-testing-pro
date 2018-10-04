---
ID: 7463
post_title: 'Sysdig Secure 1.5 &#8211; enhanced kubernetes security &#038; compliance; integrates with Google Cloud.'
author: Knox Anderson
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-secure-1-5-enhanced-kubernetes-security-integrates-with-google-cloud/
published: true
post_date: 2018-04-27 15:15:29
---
The Sysdig Secure 1.5 release furthers our goal of unifying security, performance monitoring, and forensics. This release includes a new UI and workflows to make security & compliance accessible to application developers as well as security teams. Sysdig Secure 1.5 brings feedback about what AppDev’s, Platform Operators, and Security Operations teams want: An easy way to configure policy, detect events, and mitigate threats as quickly as possible. 



## Updated Look & Feel of the Sysdig Intelligence Platform {#updatedlookfeelofthesysdigintelligenceplatform}

We’ve updated the interface of both Sysdig Monitor and Sysdig Secure to bring a more modern look and feel to our products, while also providing more real estate within the app to what matters most: Your data. Switching between monitor and secure is just a click and any grouping configuration, notification channels, or integration work is automatically shared and can be used across products.

![Sysdig Secure Screenshot][1]

## Kubernetes Security for the App Developer {#kubernetessecurityfortheappdeveloper}

Sysdig Secure allows users to scope policies by any piece of relevant metadata in their environment. If an app developer is responsible for a specific image, or an orchestrated service they can easily apply a single policy to protect all the containers associated with a given label or tab. The same goes for Platform Operators or SecOps teams who need to configure policies to an availability zone or Security group for compliance reasons. 

![Sysdig Secure Policies][2]

*In this screenshot you can see the different kubernetes deployment options that can be used to scope policies to help protect containers at scale.*

Besides being able to scope policies by metadata our customers wanted a simple way to configure policies so that everyone in the organization can understand them. This falls in nicely with the DevSecOps mantra of *you build it, you secure it, you run it.* Our new enhanced policy rules let you easily whitelist and blacklist processes, container images, files, ports, and more, while still allowing you to cover advanced scenarios with Falco based rules.

![image alt text][3]

*This screenshots shows all the different policies configured based on scope in your environment. The policy editor shows how users can easily limit inbound/outbound connections or detect unwanted activity on a range of ports.*

## Default Policies mapped to Compliance Frameworks {#defaultpoliciesmappedtocomplianceframeworks}

Sysdig Secure makes it easier to ensure that your organization is meeting its compliance regulations in many ways. One new method by which we achieve this is by mapping our default policies to common compliance regulations. That means the secops team or a developer can more easily understand why a rule is being applied, and if a violation occurs, what the potential impact of that violation is.

Sysdig secure comes with dozens of out-of-the-box policies today, that typically cover 90%+ of an enterprise’s needs. With the new policy editor, creating that last 10% is a simple task for any cybersecurity or devops professional.

## Increased Investigation efficiency {#increasedinvestigationefficiency}

If a policy condition is met and an event is triggered a security analyst must quickly understand if the event is malicious activity, if the existing policy must be tuned, or if there is no issue. We’ve expanded our event feed to group related events in the same timeframe as well as provide a robust summary to make the event classification process easier.

All events are also enriched with full kubernetes context, meaning no additional manual log correlation is needed to find out all the services a particular image may relate to. 

![image alt text][4]

*Within the details section a user can quickly see full details about why a policy violation occurred and how it can relate to the rest of your environment. Here we can quickly see the connect was attempted on port 81 by the nginx - g daemon off; command.*

## Sysdig Secure + Cloud Security Command Center {#sysdigsecurecloudsecuritycommandcenter}

Google’s Cloud Security Command Center helps security teams gather data, identify threats, and act on them before they result in business damage or loss. 

In this latest release we’ve added an integration between [Sysdig Secure and the Cloud Security Command Center][5]. Sysdig events will be correlated to 2 kinds of resources from Google Cloud Platform: compute instances, either launched manually or part of a GKE instance pool, and also container images from the <a href="https://cloud.google.com/container-registry/" target="_blank">Google Container Registry</a>.

Sysdig Secure enriches all events from containers with all the relevant instance, container image, Kubernetes scope, and cloud metadata. To make events more meaningful to Kubernetes Security teams add the different custom Kubernetes properties from Sysdig into the findings table, including cluster id, namespace, deployment or pod.

Check out what customers are saying about the integration:

"We chose to develop on Google Cloud for its robust, cost-effective platform. Sysdig is the perfect complement because it allows us to effectively secure and monitor our Kubernetes services with a single agent," said Ashley Penny, VP of infrastructure, Cota Healthcare. "We're excited to see that Google and Sysdig are deepening their partnership through this product integration."

![image alt text][6]

*Sysdig Secure enriches all events from containers with all the relevant instance, container image, Kubernetes scope, and cloud metadata. To make events more meaningful to Kubernetes Security teams add the different custom Kubernetes properties from Sysdig into the findings table, including cluster id, namespace, deployment or pod.*

Conclusion

If you want to hear about some of the benefits directly from customers who have unified monitoring and security operations check out these case studies from <a href="https://go.sysdigrp2rs.wpengine.com/sun-run-case-study" target="_blank">Sun Run</a> and <a href="https://go.sysdigrp2rs.wpengine.com/case-study-quby" target="_blank">Quby</a>. We’re constantly releasing new features to and resources to help organizations meet compliance and security needs in their new containerized environments. 

*   [Sign-up for a trial of Sysdig Secure][7]

*   [A guide to PCI Compliance in Containers][8]

*   [Visit our Brightalk Channel for sessions on best practices securing containers][9]

 [1]: /wp-content/uploads/2018/06/sds-1.5-0.png
 [2]: /wp-content/uploads/2018/06/sds-1.5-1.png
 [3]: /wp-content/uploads/2018/06/sds-1.5-2.png
 [4]: /wp-content/uploads/2018/06/sds-1.5-3.png
 [5]: https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-for-google-cloud-security-command-center/
 [6]: /wp-content/uploads/2018/06/sds-1.5-4.png
 [7]: https://sysdigrp2rs.wpengine.com/sign-up/
 [8]: https://go.sysdigrp2rs.wpengine.com/PCI-Compliance
 [9]: https://www.brighttalk.com/channel/16287/sysdig