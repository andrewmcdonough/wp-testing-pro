---
ID: 7556
post_title: >
  GKE security with Falco and Google Cloud
  Security Command Center.
author: Néstor Salceda
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/falco-gke-kubernetes-security/
published: true
post_date: 2018-06-18 02:26:18
---
A few weeks ago, <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-for-google-cloud-security-command-center/" target="_blank" rel="noopener">we announced Sysdig partnership with Google</a> to integrate Sysdig Secure with <a href="https://cloud.google.com/security-command-center/" target="_blank" rel="noopener">Google Cloud Security Command Center</a>, a single pane of glass for your security events in Google Cloud. Today we announce that <a href="https://sysdigrp2rs.wpengine.com/opensource/falco/" target="_blank" rel="noopener">Sysdig Falco</a>, our open source project for container and Kubernetes run-time security, can also send Kubernetes security events to Google Cloud Security Command Center. Sysdig Falco is part of the underlying technology of <a href="https://sysdigrp2rs.wpengine.com/product/secure/" target="_blank" rel="noopener">Sysdig Secure</a>. Sysdig Falco is an interesting option if you are building the infrastructure layer yourself and you don't need all the enterprise features of Sysdig Secure (image scanning, reporting, incident response, UI for building security policy, audit trail, compliance, integrated forensics capabilities, etc). To have Falco forward events into Google Cloud SCC you just need to:

1.  Deploy the Sysdig Google Cloud SCC connector in your Kubernetes cluster
2.  Deploy Falco in your Kubernetes cluster and configure it to send alerts to the connector

[tweet_box design="default" float="none"]Deploy #Kubernetes and Docker runtime security integrating FOSS Sysdig #Falco with Google Cloud Security Command Center.[/tweet_box] 

## Deploy the Sysdig Google Cloud SCC connector in your Kubernetes cluster {#deploythesysdiggooglecloudsccconnectorinyourkubernetescluster}

When we presented the <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-for-google-cloud-security-command-center/" target="_blank" rel="noopener">Sysdig Google Cloud SCC connector</a>, we saw how to deploy it in Google App Engine. This time, we are going to deploy it directly in our Kubernetes cluster. We have cooked a <a href="https://github.com/draios/sysdig-gcscc-connector/blob/master/deployment.yaml" target="_blank" rel="noopener">Sysdig Google Cloud SCC connector Kubernetes manifest file</a> that deploys the connector. A *Service* exposes the application handled by a *Deployment*. A *ConfigMap* and a *Secret* handles configuration and credentials. Make sure you update them first:

    kind: ConfigMap
    apiVersion: v1
    metadata:
      name: sysdig-gcscc-connector
    data:
      org_id: "534901558763"
      project_id: cscc544401558763
      compute_project_id : arboreal-logic-197806
      compute_zone: europe-west3-a
      webhook_url: https://arboreal-logic-198306.appspot.com/events
    ---
    kind: Secret
    apiVersion: v1
    metadata:
      name: sysdig-gcscc-connector
    type: Opaque
    data:
      sysdig_token: [...]
      webhook_authentication_token: [...]
      compute_service_account_info: [...]
      security_service_account_info: [...] deployment.yaml

Notice that key values in the ConfigMap are using the same names that environment variables, but lowercased. Secret values must be base64 encoded, like this:   


    $ echo "secret_value" | base64
    

Once you have updated the YAML file with your configuration and credentials, you can go ahead and create resources in your Kubernetes with:   


    $ kubectl create -f deployment.yaml

## Deploy Falco in your Kubernetes cluster and configure the connector {#deployfalcoinyourkubernetesclusterandconfiguretheconnector}

We already showed <a href="https://sysdigrp2rs.wpengine.com/blog/runtime-security-kubernetes-sysdig-falco/" target="_blank" rel="noopener">how to deploy Falco as a daemonSet in your Kubernetes cluster</a>, so for deploying Falco here, we are going to build on those instructions. Ensure your /etc/falco/falco.yaml configuration file contains the following settings:

    json_output: true
    program_output:
      enabled: true
      keep_alive: false
      program: "\"curl -d @- -X POST --header 'Content-Type: application/json' --header 'Authorization: {{ webhook_authentication_token }}' http://sysdig-gcscc-connector.default.svc.cluster.local:8080/events/\""

This piece of configuration tells Falco will send alerts using curl to <a href="http://sysdig-gcscc-connector.default.svc.cluster.local:8080/events" target="_blank" rel="noopener">http://sysdig-gcscc-connector.default.svc.cluster.local:8080/events</a>, which is the URL where Sysdig Google Cloud SCC connector *Service* will be listening. If you have deployed connector to other URL (given by the *Namespace* and *Service* names), change that value. Also, notice that you have to replace webhook_*authentication_*token surrounded by brackets with the value that will be used the webhook authentication. If webhook_*authentication_*token is *myS3cr3tT0kenZ!*, program configuration should look like:

    json_output: true
    program_output:
      enabled: true
      keep_alive: false
      program: "\"curl -d @- -X POST --header 'Content-Type: application/json' --header 'Authorization: myS3cr3tT0kenZ!' http://sysdig-gcscc-connector.default.svc.cluster.local:8080/events/\""

## Kubernetes security events in Google Cloud Security Command Center {#kubernetessecurityeventsingooglecloudsecuritycommandcenter}

Once the two components (the connector and Falco) are configured, you will start receiving security events in Google Cloud Security Command Center. For any runtime security policy that gets triggered you will get:

*   The asset id, or where in your infrastructure the event happened
*   When did this happen
*   The Kubernetes pod name and the container id where the event was originated
*   The runtime security rule that was triggered
*   A summary of the event, as defined in the Falco rule

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/falco-gcscc-detail.png" alt="Falco GSCC Kubernetes security" class="alignleft size-full wp-image-7559" height="484" width="1000" />][1] 

To know more about Sysdig and Google, read on our partnership announcement blog post: <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-for-google-cloud-security-command-center/" target="_blank" rel="noopener">Kubernetes security for Google Cloud Security Command Center</a>.

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/falco-gcscc-detail.png