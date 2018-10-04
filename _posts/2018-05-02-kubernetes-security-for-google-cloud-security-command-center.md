---
ID: 7177
post_title: >
  Kubernetes security for Google Cloud
  Security Command Center.
author: Jorge Salamero Sanz
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-for-google-cloud-security-command-center/
published: true
post_date: 2018-05-02 22:35:32
---
Effective Kubernetes security hinges on security response teams being able to quickly detect and respond to security threats on live containers – from detection, to remediation, to forensics. With the new Sysdig container security capabilities available through Google Cloud Security Command Center, customers can view security alerts for <a href="https://cloud.google.com/kubernetes-engine/" target="_blank">Google Kubernetes Engine (GKE)</a> clusters and Google Cloud Platform (GCP) in a single pane of glass, and choose how to best take action. 

By bringing together container visibility and a native Google Kubernetes Engine integration, <a href="https://sysdigrp2rs.wpengine.com/product/secure" target="_blank">Sysdig Secure</a> provides the ability to block threats, enforce compliance, and audit activity across the infrastructure through microservice aware security policies. Security events are enriched with all container and Kubernetes metadata before being sent to the Google CSCC. This process brings relevant signals to the attention of Google Cloud customers and correlates Sysdig events with other security information sources to have a single point of view and the ability to react accordingly at all levels.

## Integrating Sysdig Secure with the Security Command Center {#integratingsysdigsecurewiththesecuritycommandcenter}

This integration receives events from Secure, adds all the relevant metadata and forwards them into Google CSCC *findings*. The integration can receive events asynchronously via a webhook notification automatically created and configured in Secure, but can also be configured to poll Secure API and send aggregated events within the last minute.

We support two deployment options:

*   The easiest is to run this as a <a href="https://cloud.google.com/appengine/" target="_blank">Google App Engine</a> application. You don't need any infrastructure requirements or additional configuration and automatically provides HTTPS endpoint for the webhook.
*   The integration is packaged as a Docker container image that you can also run as a Kubernetes pod. In this case you will have to configure your own Ingress controller with a public static IP address and TLS certificates for HTTPS.

Let's walk through App Engine deployment steps:

1\.- Clone the integration repository:

`$ git clone https://github.com/draios/sysdig-gcscc-connector/`

2\.- Expose the configuration as environment variables (I like to dump this into a <a href="https://direnv.net/" target="_blank">.envrc file</a> to export them or when using Google App Engine just configure them in the `app.yaml` file). Let's go through each configuration option:

Your Sysdig user API token can be found under `Sysdig Secure > Settings`.

    export SYSDIG_TOKEN="c6f6af86-93a5-418b-9d76-1229fcd92378"

You can get your <a href="https://cloud.google.com/resource-manager/docs/creating-managing-organization" target="_blank">Organization ID</a> with `gcloud organizations list`, the <a href="https://support.google.com/cloud/answer/6158840?hl=en" target="_blank">Project ID</a> where you have CSCC product enabled and then you must set a service account name for creating findings and the account credentials. We provide a script to generate them under `./scripts/generate_gcloud_keys`.

    export ORG_ID="534901558763"
    export PROJECT_ID="cscc544401558763" \
    export SERVICE_ACCOUNT="cscc-sa" \
    export SECURITY_SERVICE_ACCOUNT_INFO="$(cat $(pwd)/$SERVICE_ACCOUNT.json)" \

Next you will need configuration for querying the Compute instances that will be correlated with the triggered Sysdig Secure events. You will configure the Project ID where you have the monitored instances running, the service account for querying the instances IDs and the Compute zone where the instances can be found. Also here you will must set the required account credentials. Again, you can find a script under `./scripts/generate_compute_keys` to create those.

    export COMPUTE_PROJECT_ID="arboreal-logic-197806"
    export COMPUTE_SERVICE_ACCOUNT="compute-sa" \
    export COMPUTE_ZONE="europe-west3-a" \
    export COMPUTE_SERVICE_ACCOUNT_INFO="$(cat $(pwd)/$COMPUTE_SERVICE_ACCOUNT.json)" 

Finally have to define the public endpoint for receiving WebHooks, either the AppEngine URL or the configured Kubernetes ingress that routes into the connector. The WeebHook requires authentication headers although this is automatically configured on the Sysdig side by the integration, you just need to setup the the token here.

    export WEBHOOK_URL="https://arboreal-logic-198306.appspot.com/events" \
    export WEBHOOK_AUTHENTICATION_TOKEN="7a79416ed4d3776b74fcc070275ece208eeef29f" 

Once we have all the configuration ready, let's move into the next step.

3\.- Create the AppEngine project and deploy the app:

If you haven't configure the Google Cloud environment tools, run first:

`gcloud init`

Then create the app:

`gcloud app create`

And finally deploy the integration:

`gcloud app deploy`

You are done! Now all events that you configure to use the newly created Google Cloud Security Command Center notification channel, will appear on the Google side.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/sysdig_secure_webhook-1024x592.png" alt="Sysdig Secure webhook for GSCC" width="1024" height="592" class="aligncenter size-large wp-image-7179" />][1]

[tweet_box design="default" float="none"]How to get #Kubernetes GKE #security events into Google Security Command Center[/tweet_box] 

## Enforcing compliance and detecting internal threats in Google Kubernetes Engine with Sysdig Secure {#enforcingcomplianceanddetectinginternalthreatsingooglekubernetesenginewithsysdigsecure}

Sysdig Secure takes a services-aware approach to container security & forensics. By leveraging container visibility with a native GKE integration customers can block threats, enforce <a href="https://sysdigrp2rs.wpengine.com/blog/container-pci-compliance/" target="_blank">Kubernetes compliance</a>, and audit activity across microservices. Let's now take a look at some of the different security events we'll detect that are native to Kubernetes. 

### Launch disallowed container images in kube-system {#launchdisallowedcontainerimagesinkubesystem}

Sysdig Secure provides the unique capability to scope policies by any piece of metadata associated with a host or a container. This allows organizations to configure policy for Kubernetes at the namespace, deployment or pod level. A good example of this is killing any unexpected non system container that's launched within the kube-system namespace. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/disallowed_container_in_kube-system.png" alt="Disallowed container in kube-system" width="572" height="609" class="aligncenter size-full wp-image-7181" />][2]

The policy to detect unexpected containers launched is using the scope `kubernetes.namespace.name = kube-system`. This means that the policy will apply to any container that's part of that namespace regardless of where the containers or images are physically running. We've also configured this policy to stop the launch of those unexpected containers.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/disallowed_container_in_kube-system_container_images.png" alt="Disallowed container in kube-system container images" width="572" height="175" class="aligncenter size-full wp-image-7182" />][3]

The second part of the policy is where we whitelist/blacklist the different images are allowed within the kube-system namespace. Here we can easily add a list of the different kubernetes systems containers, and then blacklist all other containers, for GKE this is:

    gcr.io/google-containers/event-exporter, gcr.io/google-containers/prometheus-to-sd, gcr.io/google-containers/fluentd-gcp, gcr.io/google_containers/heapster-amd64, gcr.io/google_containers/addon-resizer, gcr.io/google_containers/k8s-dns-kube-dns-amd64, gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64, gcr.io/google_containers/k8s-dns-sidecar-amd64,  gcr.io/google_containers/cluster-proportional-autoscaler-amd64, gcr.io/google_containers/busybox, gcr.io/google_containers/kube-proxy, gcr.io/google_containers/kubernetes-dashboard-amd64, gcr.io/google_containers/defaultbackend

Pro-tip: if you're unsure of the normal system containers check out the explore table within Sysdig Monitor to see what containers are normally running in a namespace at any point in time. 

### Detect launch of a privileged container in Kubernetes {#detectlaunchofaprivilegedcontainerinkubernetes}

Privileged containers have access to the entire host so you'll want to be notified when any are started in your environment and potentially even prevent them from running. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/detect_launch_privileged_container.png" alt="Detect launch privileged container Kubernetes" width="572" height="609" class="aligncenter size-full wp-image-7183" />][4]

There might be some reasons to launch a privileged container, like the Sysdig agent that requires that for monitoring purposes, or other infrastructure level container but this should rarely happen for applications. In this policy we've used the scope `kubernetes.daemonSet.name != sysdig-agent` and will notify users if a privileged container is started outside this. 

### Detecting hotfixes in production environments {#detectinghotfixesinproductionenvironments}

Sometimes doing a hotfix in production might be tempting for a team member. Tracking those can be really hard and lots of non-expected outcomes can happen.

Sysdig Secure can audit commands executed in the cluster via `docker exec` or `kubectl exec`, and alert of live modification of paths that should be immutable on a container image, like the binary directories.

A rule capturing any modification of the container file system can be as simple as this one:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/detect_write_below_bindir.png" alt="Detect write blow bindir" width="572" height="604" class="aligncenter size-full wp-image-7185" />][5]

And if you need to apply exception to certain paths where you have persitant storage, can be easily added through the Policy Editor UI:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/detect_write_below_bindir_paths_whitelist-blacklist.png" alt="Detect write below bindir paths whitelist blacklist" width="572" height="365" class="aligncenter size-full wp-image-7186" />][6]

## Responding to security events with the Security Command Center {#respondingtosecurityeventswiththesecuritycommandcenter}

We're excited to further our partnership with Google, but don't just take our word for it. Hear what Ashley Penny VP of Infrastructure, Cota Healthcare has to say:

"We chose to develop on Google Cloud for its robust, cost-effective platform. Sysdig is the perfect complement because it allows us to effectively secure and monitor our Kubernetes services with a single agent," said Ashley Penny, VP of infrastructure, Cota Healthcare. "We're excited to see that Google and Sysdig are deepening their partnership through this product integration."

Let's now take a look at how to use the Security Command Center to investigate and correlate events across projects, clusters, instances, and container images. 

1\.- Get oriented with the different projects, assets and sources in your account:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/google_security_command_center_dashboard_sysdig-1024x487.png" alt="Google Security Command Center dashboard with Sysdig" width="1024" height="487" class="aligncenter size-large wp-image-7187" />][7]

The Asset Inventory provides an at a glance view of the different projects you have in your account and the different assets that roll up into that project. By selecting a project you can drill in further to see the different security findings. 

2\.- Navigate through the different security findings from your environment:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/google_security_command_center_findings_sysdig-1024x487.png" alt="Google Security Command Center findings from Sysdig" width="1024" height="487" class="aligncenter size-large wp-image-7188" />][8]

The findings section will aggregate all the different security events that have happened across your infrastructure over a given time period with ability to click in for more info. Sysdig events will be correlated to 2 kinds of resources from Google Cloud Platform: compute instances, either launched manually or part of a GKE instance pool, and also container images from the <a href="https://cloud.google.com/container-registry/" target="_blank">Google Container Registry</a>.

3\.- Get Kubernetes context into the findings table:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/google_security_command_center_kubernetes_events-1024x180.png" alt="Google Security Command Center Kubernetes events" width="1024" height="180" class="aligncenter size-large wp-image-7189" />][9]

Sysdig Secure enriches all events from containers with all the relevant instance, container image, Kubernetes scope, and cloud metadata. To make events more meaningful to Kubernetes Security teams add the different custom Kubernetes properties from Sysdig into the findings table, including cluster id, namespace, deployment or pod.

4\.- Access to more details: 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/google_security_command_center_kubernetes_details-1024x506.png" alt="Google Security Command Center Kubernetes details" width="1024" height="506" class="aligncenter size-large wp-image-7190" />][10]

Clicking on a finding surfaces more attributes from GCP as well as properties passed in by Sysdig. This page also serves as a jumping off place into Sysdig Secure.

5\.- View the finding in Sysdig Secure for additional details, like commands audit of Sysdig captures for container forensics and post-mortem analysis, to find out how the possible attacker did break in into the container, what did do/execute and what data was stolen.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/sysdig_secure_event_details.png" alt="Sysdig Secure event details" width="449" height="543" class="aligncenter size-full wp-image-7191" />][11]

## Conclusion {#conclusion}

If you are running infrastructure in Google Cloud Platform, either if its your own Kubernetes cluster or the managed Google Kubernetes Engine, you need to keep an eye on your Kubernetes and application security. Sysdig Secure and Google Security Command Center working together will help you out with:

1.  **Continuous security with runtime analysis**: Certain suspicious activities, such as unexpected outgoing connections, anomalous file access or unauthorized process behavior often only come to light post deployment. With system call level instrumentation, completely transparent to the user and without any kind of container modification, Sysdig is able to provide deeper container visibility, which can be used to detect, alert, and block suspicious activity post deployment.
2.  **Less time spent manually correlating event information**: The Google Cloud SCC gives users consolidated visibility into their cloud assets and generates curated insights that provide users with a unique view of threats to their cloud assets. GCSCC integrates with a number of security tools, including Sysdig Secure, that will bring all the Kubernetes context you need to understand better your incident feed, so you can see that alongside other Google managed cloud services security events.
3.  **All encompassing forensics and post-mortem analysis for better decision making**: Sysdig records all activity, including commands, processes, network, and file system operations, enabling post-mortem analysis and forensics from the time of the attack, as well as pre-attack activity trails, everything you need for your research even if the container doesn't exist anymore because Kubernetes re-scheduled or deleted it.

We welcome all existing Google Cloud and Sysdig customers to try out this new integration and if you have any further inquiries don't hesitate to contact us through our in-chat or through <a href="https://support.sysdigrp2rs.wpengine.com/" target="_blank">Sysdig Support</a>.

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/sysdig_secure_webhook.png
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/disallowed_container_in_kube-system.png
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/disallowed_container_in_kube-system_container_images.png
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/detect_launch_privileged_container.png
 [5]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/detect_write_below_bindir.png
 [6]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/detect_write_below_bindir_paths_whitelist-blacklist.png
 [7]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/google_security_command_center_dashboard_sysdig.png
 [8]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/google_security_command_center_findings_sysdig.png
 [9]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/google_security_command_center_kubernetes_events.png
 [10]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/google_security_command_center_kubernetes_details.png
 [11]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/sysdig_secure_event_details.png