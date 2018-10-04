---
ID: 7163
post_title: >
  Active Kubernetes security with Sysdig
  Falco, NATS, and Kubeless.
author: Michael Ducy
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/active-kubernetes-security-falco-nats-kubeless/
published: true
post_date: 2018-05-02 07:17:18
---
The composability of Cloud Native platforms has dramatically changed the way we think about the management of these platforms. In this post we'll talk about how we can leverage this composability to create a more active, real-time solution to Kubernetes security. Specifically we'll show how you can detect abnormal behavior in a Kubernetes pod with Sysdig Falco, publish the detection of this abnormal behavior to a NATS messaging server, and then have Kubeless - a Serverless solution for Kubernetes - take action on the abnormal behavior.

## Current Security Challenges {#currentsecuritychallenges}

Detection and response is still one of the major challenges in the realm of security. How can you quickly detect that a system has been compromised, and mitigate the attack as quickly as possible. Typically applications are compromised, and may persist in this state for hours, days, or weeks. Additionally, attacks are beginning to focus on abusive behavior, rather than attempting to steal data, such as running cryptocurrency miners to take advantage of the compromised system's compute power. 

The ephemeral nature of Cloud Native platform also introduces some new challenges. Containers are constantly being scheduled, scaled up, scaled down, deleted, and updated. This ephemeral nature of containers means that an application or container could be compromised, the security event goes undetected, and the container eventually disappears for the reasons above.

Sysdig Falco provides the ability to shorten the detection and response cycle for security events. Falco can detect abnormal behavior, defined by a set of policies, and then notify other systems to take action. You can learn more about Falco over on the <a href="https://sysdigrp2rs.wpengine.com/opensource/falco/" target="_blank">project home page</a> or <a href="https://github.com/draios/falco" target="_blank">github repo</a>.

[tweet_box design="default" float="none"]Take action on #Kubernetes security alerts with Sysdig Falco, @nats_io, & @kubeless_sh[/tweet_box] 

## NATS {#nats}

<a href="https://nats.io/" target="_blank">NATS</a> is an open source messaging platform that focuses on providing highly scalable messaging publishing and delivery for cloud native applications, IoT applications, and microservices. NATS uses a text based message format that allowed us to easily integrate it into Falco with minimal effort. The project also provides a <a href="https://github.com/nats-io/nats-operator" target="_blank">Kubernetes operator</a> that makes it easy to stand up NATS servers on Kubernetes.

## Kubeless {#kubeless}

<a href="https://kubeless.io/" target="_blank">Kubeless</a> is an open source project from <a href="https://bitnami.com/" target="_blank">Bitnami</a> for Functions as a Service (FaaS or Serverless) on top of Kubernetes. Like NATS, Kubeless provides a <a href="https://kubeless.io/docs/quick-start/" target="_blank">Kubernetes operator</a> to quickly deploy and get started using Kubeless. There are also a wide variety of language runtimes available for Python, Node.js, Ruby, PHP, and Go. Kubeless also supports the NATS messaging server, and allows you to create function triggers based on messages received from the NATS server. This makes it easy to subscribe to Falco alerts published to NATS, and then have a Kubeless function take action on the alert. 

## Building a Solution for Active Kubernetes Security {#buildingasolutionforactivekubernetessecurity}

Falco, NATS, and Kubeless make it extremely easy to build a solution for active Kubernetes security. All three projects keep at heart the composable nature of cloud native architectures. For instance, Falco provides a generic framework for detecting abnormal system call events. It can notify another system of these events, and it's up to the receiving system to process and take action on these events. 

For this proof of concept (<a href="https://github.com/sysdiglabs/falco-nats" target="_blank">code available here</a>) we got started by following the <a href="https://kubeless.io/docs/quick-start/" target="_blank">Kubeless quickstart</a> to deploy the latest release of Kubeless, a NATS cluster, and the Kubeless NATS trigger controller. This gives us the foundational services we need to start publishing messages to NATS and trigger functions based on the messages. 

Next we wrote a <a href="https://github.com/sysdiglabs/falco-nats/blob/master/kubeless-function/delete-pod.py" target="_blank">simple Python function</a> to delete a Kubernetes Pod if a `Critical` priority event is detected. We leveraged the Kubernetes Python client, which makes it easy to interact with the Kubernetes API.

    def delete_pod(event, context):
        priority = event['data']['priority'] or None
        output_fields = event['data']['output_fields'] or None
    
        if priority and priority == "Critical"  and output_fields and output_fields['container.id'] != "host":
            name = output_fields['k8s.pod.name']
            print 'Critcal Falco alert for pod: {} '.format(name)
    
            ns=find_pod_ns(name)
            print 'Deleting POD {} in NameSpace {}'.format(name, ns)
            v1.delete_namespaced_pod(name=name, namespace=ns, body=body)

Because Falco can send alerts in JSON format, and this gets serialized into a Python dict, it's very easy to parse the Falco alert. We deployed this function using the following Kubeless command.

    kubeless function deploy --from-file delete-pod.py --dependencies requirements.txt --runtime python2.7 --handler delete-pod.delete_pod falco-pod-delete
    INFO[0000] Deploying function...                        
    INFO[0000] Function falco-pod-delete submitted for deployment 
    INFO[0000] Check the deployment status executing 'kubeless function ls falco-pod-delete'

Now that we have a deployed function, we can create the NATS trigger. This trigger will fire whenever a message is received from the NATS server with the subject (or topic) of `FALCO`, and it will call our Kubeless function we deployed above.

    kubeless trigger nats create falco-delete-pod-trigger --function-selector created-by=kubeless,function=falco-pod-delete --trigger-topic FALCO
    

We now need to get messages from Falco into the NATS server. Falco supports logging alerts to a file, stdout, syslog, or as stdin to a called program. While we could leverage the program output functionality of Falco, we wanted to minimize the number of processes running in our container, so we leveraged the logging to a file functionality. However, instead of using a normal log file, we created a Linux named pipe on a Kubernetes Volume, and sent Falco alerts to the named pipe like you would any other file. 

          containers:
            - name: falco-nats
              image: sysdiglabs/falco-nats:latest
              imagePullPolicy: Always
              volumeMounts:
                - mountPath: /var/run/falco/
                  name: shared-pipe
            - name: falco
              image: sysdig/falco:latest
              securityContext:
                privileged: true
              args: [ "/usr/bin/falco", "-K", "/var/run/secrets/kubernetes.io/serviceaccount/token", "-k", "https://kubernetes", "-pk", "-U"]
              volumeMounts:
                - mountPath: /var/run/falco/
                  name: shared-pipe
    

We then created a <a href="https://github.com/sysdiglabs/falco-nats/blob/master/falco-nats/nats-pub.go" target="_blank">small Golang program</a> to read from the named pipe, and then publish the Falco alert to NATS. This small Golang program runs inside a container in the same Pod as the Falco container, and they both mount the Kubernetes Volume containing the named pipe allowing for a publisher/consumer relationship between the two. We also used an `initContainer` to create the named pipe on the shared volume, to ensure it exists when our containers are started.

## Verifying the Setup {#verifyingthesetup}

To verify the functionality of this setup, we took the rules we published in our blog post on <a href="https://sysdigrp2rs.wpengine.com/blog/detecting-cryptojacking-with-sysdigs-falco/" target="_blank">detecting cryptominers with Falco</a>. We also have provided the vulnerable Node.js application we used in that blog post in the <a href="https://github.com/sysdiglabs/falco-nats" target="_blank">Github repo</a> for this blog. You can either exploit this application to simulate an attack, or trigger the rules by running a `kubectl exec` on the frontend Pod created by the Node.js application. 

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/falco-nats.gif" alt="Active Kubernetes Security with Falco, NATS, and Kubeless" width="800" height="487" class="aligncenter size-full wp-image-7164" />][1]

## Learning More {#learningmore}

As we've seen, the combination of Falco, NATS, and Kubeless provide a robust solution to active Kubernetes security, and provide a powerful tool in detecting and reacting to abnormal system behavior. In the world of containers, immutable infrastructure, and microservices, Falco can help ensure that deployed applications are following security best practices of Cloud Native, and if compromised, can notify other systems to take appropriate action.

Need more info? Check out the <a href="https://github.com/draios/falco/wiki" target="_blank">Falco documentation</a> on the GitHub wiki, or join the conversation on the <a href="https://slack.sysdigrp2rs.wpengine.com/" target="_blank">Sysdig Slack team</a>.

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/05/falco-nats.gif