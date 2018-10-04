---
ID: 4142
post_title: 'WordPress in Kubernetes: The Perfect Setup'
author: Jorge Salamero Sanz
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/wordpress-kubernetes-perfect-setup/
published: true
post_date: 2017-05-16 03:49:31
---
This tutorial shows how to prepare and deploy Wordpress with all required dependencies (MySQL, etc) in Kubernetes.

Why Wordpress? Wordpress is usually chosen as the de facto application when explaining how to handle a process with a server software stack.

One of the challenges Kubernetes users face is deploying and re-deploying the same components required by their applications: databases like MySQL or MariaDB, MongoDB, Redis, Memcached, Elasticsearch; frontend servers like Nginx; or just a load balancer with HAproxy, to name a few examples.

If you have a CI/CD workflow in place, you probably have Jenkins or any other similar software deploying to Kubernetes via `kubectl`: Services, Deployments, ConfigMaps, Ingress controller and maybe even persistent storage. But if you are looking for something easier and more simple, there is a Kubernetes package manager that can help you: <a href="https://helm.sh/" target="_blank">Helm</a>.

## Helm package manager and Kubernetes Charts

Helm is a Kubernetes package manager that allows you to package an application and handle all the required dependencies, including other services, and the configuration. The packages are called Charts. The idea is that instead of running `kubectl` to create the different deployments, services, etc. all the resources are defined in manifest templates that Helm installs as an atomic operation. Think of dpkg or yum, cpan or pip, but for Kubernetes apps.

<a href="https://github.com/kubernetes/charts" target="_blank">Kubernetes Charts</a> is the official repository that offers a central reference location for already packaged applications; at this time 60 apps are available in the stable branch.

### Installing Helm in OSX or Linux

As a prerequisite, you will need `kubectl` authenticated and working against your Kubernetes instance in your cloud provider or you can use <a href="https://kubernetes.io/docs/getting-started-guides/minikube/" target="_blank">Minikube</a> for testing in your local laptop.

On OSX, we recommend using Homebrew to install Helm:

    % brew install kubernetes-helm
    

On Linux, you can download the latest release and unpack it:

    % wget https://kubernetes-helm.storage.googleapis.com/helm-v2.2.0-linux-amd64.tar.gz
    % tar xvzf helm-v2.2.0-linux-amd64.tar.gz
    % sudo mv linux-amd64/helm /usr/local/bin/helm
    

### Init Helm in your Kubernetes cluster

Before moving along, Helm needs to be installed in your Kubernetes cluster as well. This is a very straightforward process and you just need to init Helm:

    % helm init
    $HELM_HOME has been configured at /home/bencer/.helm.
    
    
    Tiller (the helm server side component) has been installed into your Kubernetes Cluster.
    Happy Helming!
    

This will create a containerized service in Kubernetes known as Tiller. From this point on, the Helm package manager installed in your laptop will communicate with Tiller running in Kubernetes via <a href="http://www.grpc.io/" target="_blank">gRPC</a>, and Tiller will be in charge of deploying, upgrading and deleting the apps in your cluster:

    % kubectl get pods --all-namespaces
    NAMESPACE     NAME                             READY     STATUS    RESTARTS   AGE
    kube-system   kube-addon-manager-minikube      1/1       Running   0          3m
    kube-system   kube-dns-v20-qzkhp               3/3       Running   0          2m
    kube-system   kubernetes-dashboard-mtrl3       1/1       Running   0          2m
    kube-system   tiller-deploy-2923173008-n21pw   1/1       Running   0          18s
    

If you want to update the Chats database, like `apt update`, simply:

    % helm repo update
    

And if you want to upgrade Tiller:

    % helm init --upgrade
    

## Wordpress Deployment in Kubernetes

With Helm configured and Tiller up and running you can start deploying your favorite apps. For instance, let’s search for Wordpress Charts:

    % helm search wordpress
    NAME            	VERSION	DESCRIPTION                                       
    stable/wordpress	0.4.2  	Web publishing platform for building blogs and ...
    

And you can get more information about this Chart in a similar fashion to `apt-cache`:

    % helm inspect stable/wordpress
    description: Web publishing platform for building blogs and websites.
    engine: gotpl
    home: http://www.wordpress.com/
    icon: https://bitnami.com/assets/stacks/wordpress/img/wordpress-stack-220x234.png
    keywords:
    - wordpress
    - cms
    - blog
    - http
    - web
    - application
    - php
    maintainers:
    - email: containers@bitnami.com
      name: Bitnami
    name: wordpress
    sources:
    - https://github.com/bitnami/bitnami-docker-wordpress
    version: 0.4.2
    
    ---
    ## Bitnami WordPress image version
    ## ref: https://hub.docker.com/r/bitnami/wordpress/tags/
    ##
    image: bitnami/wordpress:4.7-r0
    [...]
    

Since we want our Wordpress running in its own namespace, let’s create one as we install the application:

    % helm install --namespace wordpress --name wordpress stable/wordpress                                   
    
    NAME:   wordpress
    LAST DEPLOYED: Mon Mar  6 17:38:30 2017
    NAMESPACE: wordpress
    STATUS: DEPLOYED
    
    RESOURCES:
    ==> v1/Service
    NAME                 CLUSTER-IP      EXTERNAL-IP  PORT(S)                     AGE
    wordpress-wordpress  10.109.108.55   <pending>    80:32435/TCP,443:30044/TCP  2s
    wordpress-mariadb    10.102.199.160  <none>       3306/TCP                    1s
    
    ==> extensions/v1beta1/Deployment
    NAME                 DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
    wordpress-mariadb    1        1        1           0          1s
    wordpress-wordpress  1        1        1           0          1s
    
    ==> v1/Secret
    NAME                 TYPE    DATA  AGE
    wordpress-wordpress  Opaque  2     2s
    wordpress-mariadb    Opaque  2     2s
    
    ==> v1/ConfigMap
    NAME               DATA  AGE
    wordpress-mariadb  1     2s
    
    ==> v1/PersistentVolumeClaim
    NAME                           STATUS  VOLUME                                    CAPACITY  ACCESSMODES  AGE
    wordpress-wordpress-apache     Bound   pvc-551b2739-028b-11e7-ad79-12182a6edca4  1Gi       RWO          2s
    wordpress-wordpress-wordpress  Bound   pvc-551b9867-028b-11e7-ad79-12182a6edca4  8Gi       RWO          2s
    wordpress-mariadb              Bound   pvc-551c0c25-028b-11e7-ad79-12182a6edca4  8Gi       RWO          2s
    
    
    NOTES:
    1. Get the WordPress URL:
    
      NOTE: It may take a few minutes for the LoadBalancer IP to be available.
            Watch the status with: 'kubectl get svc --namespace wordpress -w wordpress-wordpress'
    
      export SERVICE_IP=$(kubectl get svc --namespace wordpress 	wordpress-wordpress -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
      echo http://$SERVICE_IP/admin
    
    2. Login with the following credentials to see your blog
    
      echo Username: user
      echo Password: $(kubectl get secret --namespace wordpress wordpress-wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode)
    </none></pending>

You can find all the <a href="https://github.com/kubernetes/charts/tree/master/stable/wordpress" target="_blank">available configuration options here</a>, and you can set these either using the command line `--set` or in a YAML file. Also, all Charts output some instructions on how to proceed after the package is installed. In this case, you should see how to get the Wordpress Ingress controller URL with:

    % kubectl get svc --namespace wordpress wordpress-wordpress -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
    

Note: using `.status.loadBalancer.ingress[0].ip` didn’t work for me as expected an IP address and I had a hostname instead, but you can always inspect the *wordpress* service with `kubectl` to get full details:

    % kubectl describe svc wordpress-wordpress --namespace wordpress        
    Name:			wordpress-wordpress
    Namespace:		wordpress
    Labels:		        app=wordpress-wordpress
    			chart=wordpress-0.4.2
    			heritage=Tiller
    			release=wordpress
    Selector:		app=wordpress-wordpress
    Type:			LoadBalancer
    IP:			10.109.108.55
    LoadBalancer Ingress:	a55201964028b11e7ad7912182a6edca-1068550576.us-east-1.elb.amazonaws.com
    Port:			http	80/TCP
    NodePort:		http	32435/TCP
    Endpoints:		192.168.224.67:80
    Port:			https	443/TCP
    NodePort:		https	30044/TCP
    Endpoints:		192.168.224.67:443
    Session Affinity:	None
    Events:
      FirstSeen	LastSeen	Count	From			SubObjectPath	Type		Reason			Message
      ---------	--------	-----	----			-------------	--------	------			-------
      9m		9m		1	{service-controller }			Normal		CreatingLoadBalancer	Creating load balancer
      9m		9m		1	{service-controller }			Normal		CreatedLoadBalancer	Created load balancer
    

So just head to: *https://a55201964028b11e7ad7912182a6edca-1068550576.us-east-1.elb.amazonaws.com* and you will be able to access your new Wordpress blog.

You can also check the administrator user credentials that are stored in a Kubernetes secret:

    % kubectl get secret --namespace wordpress wordpress-wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode
    

And if you want to see this information at any point, you can use `helm status wordpress` to show it again.

Helm can install and handle multiple apps, and can list the different apps installed:

    % helm ls
    NAME     	REVISION	UPDATED                 	STATUS  	CHART          	NAMESPACE  
    sysdig   	1       	Mon Mar  6 12:40:22 2017	DEPLOYED	sysdig-0.2.0   	default    
    wordpress	1       	Mon Mar  6 12:48:11 2017	DEPLOYED	wordpress-0.4.2	wordpress
    

And you can also delete them once you have finished with `helm delete`.

## Monitoring Wordpress in Kubernetes

Along with your app stack, monitoring with telemetry and performance metrics for troubleshooting, dashboards and alerts is a must. Sysdig Monitor is probably your best bet for doing all this with one tool in Kubernetes and in less than 5 minutes!

Sysdig Monitor agent has always been very easy to install, either running the plain Docker command or deploying it using a DaemonSet, but now <a href="https://github.com/kubernetes/charts/tree/master/stable/sysdig" target="_blank">Sysdig can also be deployed with Helm</a> since it’s available in the official Kubernetes Charts repository.

You just need to run the following command, including the *AccessKey*, which you will find in <a href="https://app.sysdigcloud.com/#/settings/user" target="_blank">your Sysdig Monitor user account settings</a>.

    % helm install --namespace sysdig --set sysdig.AccessKey="YOUR-ACCESS-KEY",sysdig.AgentTags="cluster:kubernetes-dev" stable/sysdig
    NAME:   needled-quoll
    LAST DEPLOYED: Mon Mar  6 16:02:49 2017
    NAMESPACE: sysdig
    STATUS: DEPLOYED
    
    RESOURCES:
    ==> v1/Secret
    NAME                  TYPE    DATA  AGE
    needled-quoll-sysdig  Opaque  2     1s
    
    ==> extensions/v1beta1/DaemonSet
    NAME                  DESIRED  CURRENT  READY  NODE-SELECTOR  AGE
    needled-quoll-sysdig  3        3        0      <none>         1s
    
    NOTES:
    Sysdig Monitor agents are spinning up on each node in your cluster. After a few seconds, you should see your hosts appearing in the Explore tab:
    
        https://app.sysdigcloud.com/#/explore/overview/l:10
    
    No further action should be required.
    </none>

That’s it.

Now you can list both Charts running:

    % helm list                              
    NAME         	REVISION	UPDATED                 	STATUS  	CHART          	NAMESPACE  
    needled-quoll	1       	Mon Mar  6 16:02:49 2017	DEPLOYED	sysdig-0.2.0   	sysdig     
    wordpress    	1       	Mon Mar  6 15:28:10 2017	DEPLOYED	wordpress-0.4.2	mywordpress
    

To start monitoring your Kubernetes deployment, go to Sysdig Monitor and from there you can visualize your deployment topology, configure dashboards and set up alerts. Through the Explore tab you can browse your infrastructure. Not only can you see hosts and containers running on them, you can also group containers using Kubernetes logic: *cluster > namespaces > deployments or services > pods and containers*.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/wordpress_kubernetes_monitor-1024x481.png" alt="Wordpress Kubernetes Monitor Dashboard" width="1024" height="481" class="aligncenter size-large wp-image-4143" />][1]

Thanks to Sysdig’s ability to autodiscover the services running in your containers you will automatically get views of your Wordpress containers, including application-layer metrics such as: HTTP requests and HTTP error count per second, average and max response time for each microservice, and also which are the top requested URLs or which are the slowest ones!

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/wordpress_kubernetes_http_response_time-1024x484.png" alt="Wordpress Kubernetes Response Time" width="1024" height="484" class="aligncenter size-large wp-image-4144" />][2]

## Recommended alerts for Wordpress in Kubernetes

<a href="https://sysdigrp2rs.wpengine.com/blog/alerting-kubernetes/" target="_blank">Monitoring any service running in Kubernetes</a> is a little more tricky that monitoring the same service in a bare metal static infrastructure. Now you have to monitor not only the host and the services but also the containers and the orchestration platform. There is no rule that fits all scenarios, but we are going to throw out a few suggestions here of what should probably be in your alerts list.

### Host-level alerts

Is any of your hosts down? If its a minion it's probably not too bad because Kubernetes will reschedule the containers somewhere else. If it’s your master and you only have one you will probably want to know ASAP.

You might want to add some other host-level resource consumption alerts like disk usage over 80%, if you have been swapping for a few minutes and the load is way above normal. But remember, it's not worth waking you up if there isn’t anything broken.

### Kubernetes alerts

This is the new layer and basically you want to make sure the resources defined by Kubernetes are running. For our Wordpress example, I’m pretty sure that everyone would like to know if there are no containers running for a given service, which should never happen!

    kubernetes.replicaSet.replicas.running < 1
    </code>

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/wordpress_kubernetes_alert.png" alt="Wordpress Kubernetes Alert" width="510" height="552" class="aligncenter size-full wp-image-4145" />][3]

But also, you will want to know if the number of container running is lower that the number of replicas you wanted. This can be triggered with the following condition:

    kubernetes.replicaSet.replicas.running < kubernetes.replicaSet.replicas.desired
    </code>

This should be applied to both the *wordpress* and *mysql* service.

Just making sure the containers are alive is not enough; you need to make sure they are not being continuously restarted because the process running inside is crashing. This is known as *CrashLoopBackOff*, and you can receive an alert when this happens using the restart count metric or an alert on the event:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/wordpress_kubernetes_alert-1.png" alt="Wordpress Kubernetes Alert Event" width="510" height="552" class="aligncenter size-full wp-image-4146" />][4]

We will apply this alert across everything in your cluster, making it trigger for each pod that matches this condition: *kubernetes.pod.restart.count* > 4 over the last 2 minutes.

### Service level alerts

You still need to make sure that your services are running: that the Wordpress webserver is responding to HTTP requests properly and queries are going through the MySQL database. Defining alerts at this level depends heavily on your traffic patterns. If you have a regular, sustained number of visits it is easy to set a minimum threshold of HTTP requests or MySQL queries, but if your traffic is random these kind of alerts will trigger many false positives.

In general, alerting should be configured using your working metrics (those that are a clear indicator that your app is working), triggering notifications to show that something has stopped working. Still, in most cases, you should set up a few alerts based on infrastructure availability or resources that always need to be there. If you want to learn more about Kubernetes alerting, don’t miss our <a href="https://sysdigrp2rs.wpengine.com/blog/alerting-kubernetes/" target="_blank">Monitoring Kubernetes (part 2): Best practices for alerting on Kubernetes</a>.

## Next steps

We think Helm is great for quickly and easily deploying the services you need for running your app in Kubernetes. Now you can also deploy the <a href="https://sysdigrp2rs.wpengine.com/product/" target="_blank">Sysdig Monitor</a> agent with Helm too. If you want to close the circle you can create your own Chart package for your app and host it in your private repository, with everything still handled by Helm.

If you are looking at an even more user-friendly way to expose Helm to your Kubernetes users, check out <a href="https://github.com/helm/monocular" target="_blank">Monocular</a>, a Helm UI that you can also self-host in your Kubernetes.

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/wordpress_kubernetes_monitor.png
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/wordpress_kubernetes_http_response_time.png
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/wordpress_kubernetes_alert.png
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/05/wordpress_kubernetes_alert-1.png