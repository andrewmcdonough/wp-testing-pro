---
ID: 5442
post_title: 'Kubernetes Security: How to harden internal kube-system services'
author: Mateo Burillo
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-harden-kube-system/
published: true
post_date: 2017-11-27 04:33:28
---
Are you looking at how to improve your Kubernetes security?. We have put together here the best practices for implementing run-time security on the kube-system components (kubelet, apiserver, scheduler, kubedns, etc) deployed in Docker containers.

One of the main sources of concern for companies approaching the container paradigm has traditionally been **security**. It's a radical infrastructure switch after all, and certain level of caution is perfectly healthy.

Kubernetes devs are aware of this and the platform has improved leaps and bounds in this respect. Work on the <a href="https://kubernetes.io/docs/admin/authorization/rbac/" target="_blank">RBAC API</a>, integrated secrets vault or certificate rotation mechanisms are the latest examples of this effort.

Kubernetes security is a vast topic already, with several guidelines written by the <a href="https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/" target="_blank">project itself</a> and many third parties. These guides focus on tuning cluster configuration parameters, restricting user privileges or secret management. But what if somebody finds a way to bypass those? Or a software component does something unexpected because it suffers from a bug or security vulnerability that your static scanning will never catch?

Run-time security provides an extra layer of protection for those times malicious users or software behave in a way you didn't prepare for.

In this article we are going to create and test Kubernetes run-time security policies using <a href="https://sysdigrp2rs.wpengine.com/product/secure/" target="_blank">Sysdig Secure</a>, our container native security and forensics product.

[tweet_box design="default" float="none"] Implement run-time #security policy for your #Kubernetes components: kubelet, apiserver, scheduler, kubedns, etcd & kube-proxy, #securebydefault [/tweet_box] 

Let's describe the kube-system components that we are going to consider for this article.

## kube-system security: Core components {#kubesystemsecuritycorecomponents}

Kubernetes clusters can have different building blocks, some of them are optional like the dashboard, your preferred service mesh broker, or a SDN implementation but these are the core components that you will find everywhere and we are going to cover here:

**kube-apiserver**: The central communications hub of the cluster. Provides REST endpoints to interact with the other cluster entities and stores the distributed state in the etcd backend.

**etcd**: The database backend where the cluster configuration, state and related information persists.

**kube-controller-manager**: This component implements the main control loop, in other words, it observes the differences between the current and desired cluster states and performs the changes needed to move towards desired state. When you launch a *ReplicationController*, it gets included in this component.

**kube-scheduler**: Watches newly created pods that have no node assigned, and selects a node for them to run on. From version 1.6 onwards, you can plug in your own <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-scheduler/" target="_blank">custom Kubernetes scheduler</a>.

**kube-dns**: Unsurprisingly, an internal cluster DNS server. Kube DNS will automatically configure registers for Kubernetes namespaces, services and pods. This way the pods can easily locate other services in the cluster.

**kubelet**: The cluster agent that runs on every Kubernetes node. The kubelet launches the pods using the available container engine (Docker, rkt, etc) and periodically checks and reports pod status.

**kube-proxy**: Another service that runs on every node, providing the necessary network translation between service endpoints and pods.

The kubelet and kube-proxy run as processes directly in the Kubernetes nodes, the other components are typically run as cluster Docker containers inside the *kube-system* namespace.

## Kubernetes security by default with Sysdig Secure {#kubernetessecuritybydefaultwithsysdigsecure}

We are going to produce a security ruleset for these components using a whitelisting approach (explicitly declaring valid entities, banning everything else). The container/microservice paradigm goes very well with this approach because containers are already minimal and predictable by design.

### Sysdig Secure {#sysdigsecure}

Sysdig Secure, part of the <a href="https://sysdigrp2rs.wpengine.com/" target="_blank">Sysdig Container Intelligence Platform</a>, is designed to provide container run-time security & forensics for enterprises. Sysdig Secure's deep container visibility provides insight into what’s happening inside the containers and leverages key orchestration technologies like Kubernetes, Docker, OpenShift, Amazon ECS to bring in metadata and apply rules from a service and application perspective.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/Explore-Policy-Violation.png" alt="Sysdig Secure overview" width="2556" height="1280" class="alignleft size-full wp-image-5450" />][1]

The Sysdig Secure rule syntax is the same used in Sysdig Falco, and is documented <a href="https://github.com/draios/falco/wiki" target="_blank">here</a>. Anyways, most of the rules that we are going to use are quite self-explanatory.

### kube-system process security {#kubesystemprocesssecurity}

One of the easiest and most effective whitelist that you can configure is the list of allowed processes. This task that would be somewhat tedious and error prone in a classical all-purpose server, but it's quite straightforward for microservices.

For example, let's see the processes on our etcd service container:

    $ kubectl exec -it etcd --namespace=kube-system sh
    / # ps aux
    PID   USER     TIME   COMMAND
        1 root      34:42 etcd --listen-client-urls=http://127.0.0.1:2379 --advertise-client-urls=http://127.0.0.1:2379 --data-dir=/var/lib/etcd
       41 root       0:00 sh
       45 root       0:00 ps aux

So, basically just the etcd process, anything else would be extremely suspicious.

Let's take a look to the API server:

    $ kubectl exec -it kube-apiserver --namespace=kube-system sh
    / # ps aux
    PID   USER     TIME   COMMAND
        1 root     117:34 kube-apiserver --insecure-port=0 --allow-privileged=true --requestheader-username-headers=X-Remote-User --service-cluster-ip-range=10.96.0.0/12 --proxy-client
       40 root       0:00 sh
       44 root       0:00 ps aux

Similarly, a single process running here.

So, you can write a Sysdig Secure list and rule similar to this one:

    - list: etcd_authorized_processes
      items: [etcd]
    
    - rule: Etcd allowed processes
      desc: Whitelist of authorized etcd processes
      condition: spawned_process and not proc.name in etcd_authorized_processes
      output: Unauthorized process (%proc.cmdline) running in (%container.id)
      priority: ERROR

And the following Kubernetes security policy:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/sysdig_secure_kube_system_security_rule.png" alt="Sysdig Secure Kube-system security rule" width="2154" height="896" class="alignleft size-full wp-image-5449" />][2]

We wrote already on <a href="https://sysdigrp2rs.wpengine.com/blog/sysdig-secure-docker-run-time-security/" target="_blank">Sysdig Secure: Docker run-time security</a> about the policy creation process more in detail. In a nutshell: you are restricting the scope of this rule to `kubernetes.namespace.name=kube-system` and Docker containers with the label `component=etcd`. This security violation is critical enough to stop the container immediately (see *Actions*) and of course, on a real scenario you will configure several notification channels, typically email or Slack but you can also go for webhooks, VictorOps, PagerDuty, etc.

Let's trigger it the policy we just created, opening a shell ino the etcd container and running any process, for example:

    / # ls
    bin   dev   etc   home  proc  root  sys   tmp   usr   var
    / # user@localhost:~/kubernetes$

Notice that I have been automatically expelled from the container. The container where I was running that command has been inmediately killed by Sysdig.

You will get the event in your Sysdig Secure stream and clicking on it you will be able to see the detail (just including the sections relevant for this example below):

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/kubernetes_security_alert_feed.png" alt="Kubernetes security alert feed" width="973" height="74" class="alignleft size-full wp-image-5448" />][3]

     ...
    Severity
     High
    
    Container
    ID: f1600721a783
     Name: k8s_etcd_etcd-ip..._..._kube-system_d76e26fba3bf2bfd215eb29011d55250_0
     Image: gcr.io/google_containers/etcd-amd64@sha256:d83d3545e06fb035db8512e33bd44afb55dea007a3abd7b17742d3ac6d235940
    
    Details
     1 unique outputs were generated across the policy events in this group (Zoom into event group to see all occurrences):
     Unauthorized process (ls ) running in (f1600721a783)
     ...

For the sake of brevity, we are only going to include the YAML rules for the next sections, as you have seen, creating and testing the corresponding Sysdig Secure policies is a straightforward process.

### Trusted containers in Kubernetes kube-system {#trustedcontainersinkuberneteskubesystem}

In conjunction with other tools to run your own images registry, Sysdig Secure offers an additional layer of run-time security against the use of untrusted containers. This is especially important for the `kube-system` namespace where the allowed list of pods is pretty small and immutable. The following rule will automatically detect (and kill) any container that doesn't come from any of the allowed images:

    - macro: allowed_containers
      condition:  (container.image startswith gcr.io/google_containers/etcd-amd64 or
                  container.image startswith gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64 or
                  container.image startswith gcr.io/google_containers/k8s-dns-kube-dns-amd64 or
                  container.image startswith gcr.io/google_containers/k8s-dns-sidecar-amd64 or
                  container.image startswith gcr.io/google_containers/kube-apiserver-amd64 or
                  container.image startswith gcr.io/google_containers/kube-controller-manager-amd64 or
                  container.image startswith gcr.io/google_containers/kube-proxy-amd64 or
                  container.image startswith gcr.io/google_containers/kube-scheduler-amd64)
    
    - rule: Unallowed container running in kube-system namespace
      desc: Unallowed container running in kube-system namespace
      condition: container and not allowed_containers
      output: Unallowed container running in kube-system namespace (%container.info)
      priority: ERROR

### Kubernetes secrets security: process tries to read a secret after boot {#kubernetessecretssecurityprocesstriestoreadasecretafterboot}

Kubernetes has a <a href="https://kubernetes.io/docs/concepts/configuration/secret/" target="_blank">secrets mechanism</a> to securely initialize pods with artifacts like private keys, passwords, tokens, etc. Generally, a pod will need to access its secrets during start up. You can detected if a long running process tries to access a secret file.

Assuming you mount your secrets under `/etc/secrets` (replace with your configured path) this rule will detect and react when a process in a container reads those sensible files:

    - macro: proc_is_new
      condition: proc.duration <= 5000000000
    
    - rule: Read secret file after startup
      desc: >
        an attempt to read any secret file (e.g. files containing user/password/authentication
        information) Processes might read these files at startup, but not afterwards.
      condition: fd.name startswith /etc/secrets and open_read and not proc_is_new
      output: >
        Sensitive file opened for reading after startup (user=%user.name
        command=%proc.cmdline file=%fd.name)
      priority: WARNING

### Pods trying to connect to their local kubelet {#podstryingtoconnecttotheirlocalkubelet}

In a Kubernetes cluster, the API server talks to the kubelets and these set up the pods in each node. Usually, the pods shouldn't connect to the kubelet or try to scrape its metrics unless you have a service that explicitly needs that. Let's fire an alarm if we detect this behavior.

The API server connects to the kubelet service using port 10250. 10255 and 10248 (now deprecated) are read-only health check and stats ports. To detect any pod connecting to the local kubelet, we will use a rule like this:

    - macro: kubelet_ports
     condition: fd.sport in (10248, 10250, 10255)
    
    - rule: Pod connecting to kubelet
     desc: A pod is opening an outbound network connection to the local kubelet
     condition: outbound and fd.sip = "127.0.0.1" and kubelet_ports
     output: Pod connecting to local kubelet (command=%proc.cmdline %container.info connection=%fd.name)
     priority: WARNING

### kube-system user security policies {#kubesystemusersecuritypolicies}

Another common attack symptom could be the modification of attributes and permissions of the default users to gain access to privileged information (forging an URL for example).

These kube-system components do not need to modify or create any new user, you can detect and alert if any container tries to perform these actions.

Conveniently enough, there are already lists and macros for the user management tools. The following rule makes it super easy:

    - rule: User management operations
      desc: User created or modified inside a pod
      condition: proc.name in (user_mgmt_binaries) and not proc.pname in (cron_binaries, systemd, run-parts)
      output: User management binary run on container (command=%proc.cmdline %container.info)
      priority: ERROR

Let's test this policy:

    $ kubectl exec -it kube-dns-545bc4bfd4-zm282 --namespace=kube-system sh
    Defaulting container name to kubedns.
    Use 'kubectl describe pod/kube-dns-545bc4bfd4-zm282' to see all of the containers in this pod.
    / # passwd postgres

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/kubernetes_security_user_management.png" alt="Kubernetes security user management" width="1105" height="77" class="alignleft size-full wp-image-5447" />][4]

In this case we have been looking for user related changes but the default ruleset of Sysdig Secure already detects any write below */etc*, this offers you extra protection in case the attacker tries to change the valid sources of users, for example modifying */etc/nsswitch.conf* ;-).

### Kubernetes file system security: write-allowed directories {#kubernetesfilesystemsecuritywritealloweddirectories}

Most container directories are generally static and read-only. The list of write-allowed directories should be easy to define and any violation immediately detected. This is an example to detect writes outside the allowed path:

    - macro: etcd_write_allowed_directories
      condition: evt.arg[1] startswith /var/lib/etcd
    
    - rule: Write to non write allowed dir (etcd)
      desc: attempt to write to directories that should be immutable
      condition: open_write and not etcd_write_allowed_directories
      output: "Writing to non write allowed dir (user=%user.name command=%proc.cmdline file=%fd.name)"
      priority: ERROR

### Kubernetes network security: processes opening a listening port {#kubernetesnetworksecurityprocessesopeningalisteningport}

Other interesting whitelist is the list of processes allowed to open a listening connection. For example, in this rule we only let kube-proxy to open listening ports:

    - list: kube_proxy_port_processes
      items: [kube-proxy]
    
    - rule: Unauthorized process opened a port
      desc: A kube_proxy process tried to open a port and is not whitelisted
      condition: evt.type=listen and not proc.name in (kube_proxy_port_processes)
      output: Non-whitelisted process opened a port (command=%proc.cmdline connection=%fd.name)
      priority: WARNING

### Kubernetes network security: Processes opening an outbound connection {#kubernetesnetworksecurityprocessesopeninganoutboundconnection}

The list of processes that can initiate a connection from your kube-system pods should be fairly limited as well, let's take advantage of that. For example here, only kube-apiserver can initate an outbound connection:

    - list: kube_apiserver_outbound_processes
      items: [kube-apiserver]
    
    - rule: Unauthorized process opened an outbound connection
      desc: A kube-apiserver process tried to open an outbound connection and is not whitelisted
      condition: outbound and not proc.name in (kube_apiserver_outbound_processes)
      output: Non-whitelisted process opened an outbound connection (command=%proc.cmdline connection=%fd.name)
      priority: WARNING

Let's try it:

    $ kubectl exec -it kube-apiserver --namespace=kube-system sh
    / # wget www.google.com

Alert description text:

    Non-whitelisted process opened an outbound connection (command=wget www.google.com connection=10.0.11.228:41636->172.217.13.68:80)

### Kubernetes network security: Detecting NodePort endpoints {#kubernetesnetworksecuritydetectingnodeportendpoints}

A human error or maybe a malicious user can configure a service as type *Nodeport*, thus bypassing the firewalls and other security measures that you have configured for your load balancers:

30000 to 32767 is the default port range in Kubernetes for <a href="https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport" target="_blank">NodePort services</a>. This is a rule to detect that:

    - rule: Unexpected NodePort connection
      desc: A service has been declared using type NodePort
      condition: (outbound or inbound) and fd.sport >= 30000 and fd.sport <= 32767
      output: A service is using a NodePort connection (command=%proc.cmdline connection=%fd.name)
      priority: WARNING

### Default Sysdig Secure rules and policies {#defaultsysdigsecurerulesandpolicies}

In addition to all the security rules and policies that we just listed, the default Sysdig Secure policies had been designed to protect containers from several common threats and attacks. Most of them will be useful to protect your kube-system out of the box, or with some minor customizations.

You will find policies like:

*   **Modify binary dirs**: an attempt to modify any file below a set of binary directories.
*   **Change thread namespace**: an attempt to change a program/thread namespace (commonly done as a part of creating a container) by calling setns.
*   **Launch Sensitive Mount Container**: detect the initial process started by a container that has a mount from a sensitive host directory (i.e. /proc). Exceptions are made for known trusted images.
*   **Launch Privileged Container**: detect the initial process started in a privileged container. Exceptions are made for known trusted images.
*   **System procs network activity**: any network activity performed by system binaries that are not expected to send or receive any network traffic.

## Auto generate Kubernetes security rules and policies {#autogeneratekubernetessecurityrulesandpolicies}

Creating all these whitelisting rules by hand can be laborious, you can auto generate the rules that will be used as base for your policies using the following YAML format:

<script src="https://gist.github.com/0c4bd458adca8d8bbc007569ed5da4ad.js"></script> <noscript>
  <pre><code>
File: input_rules.yaml
----------------------

- podname: etcd
  proc: [etcd]
  write_dir: [/var/lib/etcd]
  outbound_proc: [etcd]
  listen_proc: [etcd]

- podname: kube_apiserver
  proc: [kube-apiserver]
  write_dir: false
  outbound_proc: [kube-apiserver]
  listen_proc: [kube-apiserver]

- podname: kube_dns
  proc: [dnsmasq, dnsmasq-nanny, sidecar, kube-dns]
  write_dir: [/var/run/dnsmasq.pid, /dev/null]
  outbound_proc: [kube-dns]
  listen_proc: [kube-dns, sidecar, dnsmasq]

- podname: kube_controller
  proc: [kube-controller-manager]
  write_dir: false
  outbound_proc: [kube-controller-manager]
  listen_proc: [kube-controller-manager]

- podname: kube_scheduler
  proc: [kube-scheduler]
  write_dir: false
  outbound_proc: [kube-scheduler]
  listen_proc: [kube-scheduler]
</code></pre>
</noscript>

This input file will auto generate the mentioned whitelisting rules for processes, write-allowed directories, process allowed to start outbound connections and process allowed to open listening ports.

And this <a href="https://gist.github.com/mateobur/61b86311cee43e596f1a06d725f16a04" target="_blank">simple python script</a>.

    $ ./generate-secure-rules.py input_rules.yaml > output_rules.yaml

This will automatically generate a ruleset that you can directly copy and paste as custom rules for Sysdig Secure.

## Conclusions {#conclusions}

Strict run-time security for your kube-system pods is an effective mechanism against any attack that has already bypassed your existing security measures or that managed to exploit a new vulnerability.

Get a free <a href="https://go.sysdigrp2rs.wpengine.com/sysdig-secure-trial" target="_blank">Sysdig Secure trial</a> and protect your containers and microservices with run-time security or learn more about <a href="https://www.sysdig.org/falco/" target="_blank">Sysdig Falco</a> (single host, command line only), the open source relative of Sysdig Secure.

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/Explore-Policy-Violation.png
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/sysdig_secure_kube_system_security_rule.png
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/kubernetes_security_alert_feed.png
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/11/kubernetes_security_user_management.png