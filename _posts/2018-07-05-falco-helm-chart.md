---
ID: 7700
post_title: 'Kubernetes run-time security: Automate Sysdig Falco deployment using Helm charts.'
author: Néstor Salceda
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/falco-helm-chart/
published: true
post_date: 2018-07-05 03:14:09
---
So, you want to implement run-time security in your Kubernetes cluster? If you are looking for an open-source tool, obviously <a href="https://sysdigrp2rs.wpengine.com/opensource/falco/" target="_blank" rel="noopener">Sysdig Falco</a> is the way to go :). You can <a href="https://sysdigrp2rs.wpengine.com/blog/runtime-security-kubernetes-sysdig-falco/" target="_blank" rel="noopener">install Falco as a daemonSet</a>, but as we wanted to make things even easier and natively integrated, we have packaged Falco as a <a href="https://helm.sh/" target="_blank" rel="noopener">Helm</a> chart, the Kubernetes package manager.

[tweet_box design="default" float="none"] Deploy runtime security rules in your #Kubernetes cluster in 5 minutes using open source Sysdig #Falco and #Helm. [/tweet_box] 

## How to install Falco with Helm? {#howtoinstallfalcowithhelm}

If you are already using Helm for managing your Kubernetes applications, now you can deploy Falco in a few seconds, it only takes a simple command:

    $ helm install --name sysdig-falco-1 stable/falco

If you haven't setup Helm yet, you will have first to download the client, setup the RBAC permissions and deploy the Tiller service, all these steps are thoroughly documented in the <a href="https://docs.helm.sh/using_helm/#quickstart-guide" target="_blank" rel="noopener">Helm quickstart guide</a>, just follow the instructions there, you can complete them in a few minutes.

## Configuring the Falco Helm chart {#configuringfalcohelmchart}

Falco Helm chart exposes all the configuration settings available in Falco. You can configure Falco using variables/flags set through helm install or through the values.yaml file, the idea is that you don't need to modify the falco.yaml configuration file manually, generating more portable and repeatable deployment scripts.

For example, if you want Falco to send a message to Slack every time an anomalous behavior is detected, you would do something like this:

    $ helm install --name sysdig-falco-1 \
    --set falco.jsonOutput=true \
    --set falco.jsonIncludeOutputProperty=true \
    --set falco.programOutput.enabled=true \
    --set falco.programOutput.program="\"jq '{text: .output}' | curl -d @- -X POST [https://hooks.slack.com/services/see_your_slack_team/apps_settings_for/a_webhook_url\](https://hooks.slack.com/services/see_your_slack_team/apps_settings_for/a_webhook_url\)"\" \
    stable/falco

This command line is long and not really easy to type. There is another way to do this kind of customization: Using a custom `values.yaml` file.

Create a file with values in YAML format and name it values.yaml.

    falco:
      jsonOutput: true
      jsonIncludeOutputProperty: true
    
      programOutput:
        enabled: true
        program: "jq '{text: .output}' | curl -d @- -X POST [https://hooks.slack.com/services/see_your_slack_team/apps_settings_for/a_webhook_url\](https://hooks.slack.com/services/see_your_slack_team/apps_settings_for/a_webhook_url\)"

Then you can run:

    $ helm install --name sysdig-falco-1 -f values.yaml stable/falco

And that's it. Helm will deploy Falco in your Kubernetes, so you can keep track and version the `values.yaml` config file.

Finally, remember that all configuration flags are documented on the <a href="https://github.com/kubernetes/charts/tree/master/stable/falco" target="_blank" rel="noopener">Falco Helm chart documentation</a>.

## Generating sample events {#generatingsampleevents}

Falco Event Generator is a container that automatically generates fake anomalous activity, so you can get a taste of the kind of events that Sysdig Falco can detect. This is typically used for testing your setup, policies and notification gateways. You can install it with the fakeEventGenerator flag.

    $ helm install --name sysdig-falco-1 --set fakeEventGenerator.enabled=true stable/falco

But if you don't want to keep it running forever in your cluster, remove the chart when you are done:

    $ helm delete --purge --name sysdig-falco-1

## How to customize your Kubernetes runtime security policies using the Falco chart {#howtocustomizeyourkubernetesruntimesecuritypoliciesusingthefalcochart}

Currently Falco ships with a <a href="https://github.com/draios/falco/tree/dev/rules" target="_blank" rel="noopener">generic default ruleset</a> for detecting anomalous container activity (processes spawning a shell in a container, changes in binary path directories, etc). It's a good starting point, but probably you want to customize the run-time security rulesets or policies for the specific container images of services and applications you run.

Last month, we published <a href="https://github.com/draios/falco-extras" target="_blank" rel="noopener">default Falco runtime security rulesets</a> for the most popular Docker images like Nginx, Redis, Elasticsearch, etc or the services in kube-system, so you can implement better runtime security in your Kubernetes applications and save time, read more about it on <a href="https://sysdigrp2rs.wpengine.com/blog/docker-runtime-security/" target="_blank" rel="noopener">Implementing Docker/Kubernetes runtime security</a>.

So, how can we bring together deploying Falco using the Helm chart and additional custom Falco runtime security rulesets? Kubernetes ConfigMaps will help us here.

First of all, clone or download rules from <a href="https://github.com/draios/falco-extras" target="_blank" rel="noopener">draios/falco-extras</a> repository. We have <a href="https://github.com/draios/falco-extras/blob/master/scripts/rules2helm" target="_blank" rel="noopener">developed a little bash script</a> for helping to create a file readable by Helm with your custom rules. The Helm chart will use that file to generate the ConfigMaps that the Falco pods will mount:

    $ git clone [https://github.com/draios/falco-extras.git](https://github.com/draios/falco-extras.git)
    $ cd falco-extras
    $ ./scripts/rules2helm rules/rules-traefik.yaml rules/rules-redis.yaml > custom-rules.yaml
    $ helm install --name sysdig-falco-1 -f custom-rules.yaml stable/falco

If we read the logs from Falco pods, we will see something like:

    Tue Jun  5 15:08:57 2018: Loading rules from file /etc/falco/rules.d/rules-redis.yaml:
    Tue Jun  5 15:08:58 2018: Loading rules from file /etc/falco/rules.d/rules-traefik.yaml:

This message indicates that our custom rules have been loaded and is ready to detect anomalous activity in Traefik HTTP proxy and Redis server.

If you have, for example, a Redis deployment in your cluster you can try to execute this command:

    kubectl exec -it redis-master-0 cat /etc/passwd

The Falco pod should detect the access and generate this output:

    $ kubectl logs sysdig-falco-1-gsppx
    ...
    09:35:58.678355216: Notice Unexpected process spawned in redis container (command=cat /etc/passwd pid=9811 user=<NA> k8s.pod=redis-master-0 container=ab6769a7c1d2 image=bitnami/redis@sha256:e50375d55ea5e5912f985ae1bf8f7c95a00ec2ff7f4c18e3c9afe7b98dcdaf43) k8s.pod=redis-master-0 container=ab6769a7c1d2

## Conclusions {#conclusions}

If you are already using Kubernetes and Helm, deploying Falco is a breeze. It takes only a few seconds to install it across all your Kubernetes nodes and implement Kubernetes run-time security. If you haven't installed Helm, it just takes a few seconds more.

When we bring this together with the default <a href="https://sysdigrp2rs.wpengine.com/blog/docker-runtime-security/" target="_blank" rel="noopener">runtime security</a> rulesets, you can get an open-source end to end run-time security solution for Kubernetes in about no time. But if you don't want to build all this yourself and prefer enterprise features, customer support, etc, check out <a href="https://sysdigrp2rs.wpengine.com/product/secure/" target="_blank" rel="noopener">Sysdig Secure</a>, a commercial platform for container security, vulnerability management, compliance and forensics.