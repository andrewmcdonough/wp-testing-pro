---
ID: 6996
post_title: 'Kubernetes security context, security policy, and network policy &#8211; Kubernetes security guide (part 2).'
author: Mateo Burillo
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-psp-network-policy/
published: true
post_date: 2018-04-04 09:58:11
---
Once you have defined <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-rbac-tls/" target="_blank">Kubernetes RBAC: users and services credentials and permissions</a>, we can start leveraging Kubernetes orchestration capabilities to configure security at the pod level. In this part, we will learn how to configure security at the pod level using Kubernetes orchestration capabilities: Kubernetes Security Context, Kubernetes Security Policy and Kubernetes Network Policy resources to define the container privileges, permissions, capabilities and network communication rules. We will also discuss how to limit resource starvation with allocation management.

The 1st part of this Kubernetes Security guide focus on <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-rbac-tls/" rel="noopener" target="_blank">Kubernetes RBAC and TLS certificates</a> while part 3 goes on <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-kubelet-etcd/" rel="noopener" target="_blank">Securing Kubernetes components: kubelet, etcd and Docker registry</a>. [tweet_box design="default" float="none"]#Kubernetes all-the-#security-things: Kubernetes Security Context, Kubernetes Security Policy, Kubernetes Network Policy: who is who and how to use them[/tweet_box] 

## Kubernetes admission controllers {#kubernetesadmissioncontrollers}

An <a href="https://kubernetes.io/docs/admin/admission-controllers/" target="_blank">admission controller</a> is a piece of code that intercepts requests to the Kubernetes API server prior to persistence of the object, but after the request is authenticated and authorized. Admission controllers pre-process the requests and can provide utility functions (such as filling out empty parameters with default values), but can also be used to enforce security policies and other checks.

Admission controllers are found on the *kube-apiserver* conf file:

    --admission-control=Initializers,NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,ResourceQuota

Here are the admission controllers that can help you strengthen security:

**DenyEscalatingExec:** Forbids executing commands on an "escalated" container. This includes pods that run as privileged, have access to the host IPC namespace, and have access to the host PID namespace. Without this admission controller, a regular user can escalate privileges over the Kubernetes node just spawning a terminal on these containers.

**NodeRestriction:** Limits the node and pod objects a kubelet can modify. Using this controller, a Kubernetes node will only be able to modify the API representation of itself and the pods bound to this node.

**PodSecurityPolicy:** This admission controller acts on creation and modification of the pod and determines if it should be admitted based on the requested Security Context and the available Pod Security Policies. The PodSecurityPolicy objects define a set of conditions and security context that a pod must declare in order to be accepted into the cluster, we will cover PSP in more detail below.

**ValidatingAdmissionWebhooks:** Calls any external service that is implementing your custom security policies to decide if a pod should be accepted in your cluster. For example, you can <a href="https://github.com/kelseyhightower/grafeas-tutorial" target="_blank">pre-validate container images</a> using Grafeas, a container-oriented auditing and compliance engine, or [validate Anchore scanned images][1].

There is a recommended <a href="https://kubernetes.io/docs/admin/admission-controllers/#is-there-a-recommended-set-of-admission-controllers-to-use" target="_blank">set of admission controllers to run depending on your Kubernetes version</a>.

## Kubernetes Security Context {#kubernetessecuritycontext}

When you declare a pod/deployment, you can group several security-related parameters, like SELinux profile, Linux capabilities, etc, in a <a href="https://kubernetes.io/docs/tasks/configure-pod-container/security-context/" target="_blank">Security context</a> block:

    ...
    spec:
      securityContext:
        runAsUser: 1000
        fsGroup: 2000
    ...

You can configure the following parameters as part of your security context:

**Privileged:** Processes inside of a *<a href="https://kubernetes.io/docs/concepts/workloads/pods/pod/#privileged-mode-for-pod-containers" target="_blank">privileged</a>* container get almost the same privileges as those outside of a container, such as being able to directly configure the host kernel or host network stack.

Other context parameters that you can enforce include:

**User and Group ID for the processes, containers and volumes:** When you run a container without any security context, the 'entrypoint' command will run as root, this is easy to verify:

    $ kubectl run -i --tty busybox --image=busybox --restart=Never -- sh
    / # ps aux
    PID   USER     TIME  COMMAND
        1 root      0:00 sh

Using the *runAsUser* parameter you can modify the user ID of the processes inside a container. For example:

    apiVersion: v1
    kind: Pod
    metadata:
      name: security-context-demo
    spec:
      securityContext:
        runAsUser: 1000
        fsGroup: 2000
      volumes:
      - name: sec-ctx-vol
        emptyDir: {}
      containers:
      - name: sec-ctx-demo
        image: gcr.io/google-samples/node-hello:1.0
        volumeMounts:
        - name: sec-ctx-vol
          mountPath: /data/demo
        securityContext:
          allowPrivilegeEscalation: false

If you spawn a container using this definition you can check that the initial process is using *UID 1000*.

    USER   PID %CPU %MEM    VSZ   RSS TTY   STAT START   TIME COMMAND
    1000     1  0.0  0.0   4336   724 ?     Ss   18:16   0:00 /bin/sh -c node server.js

And any file you create inside the `/data/demo` volume will use GID 2000 (due to the *fsGroup* parameter).

**Security Enhanced Linux (SELinux):** You can assign <a href="https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.9/#selinuxoptions-v1-core" target="_blank">SELinuxOptions</a> objects using the *seLinuxOptions* field. Note that SELinux module needs to be loaded on the underlying Linux nodes for this policies to take effect.

**Capabilities:** Linux capabilities break down root full unrestricted access into a set of separate permissions. This way, you can grant some privileges to your software, like binding to a port < 1024, without granting full root access.

There is a <a href="https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities" target="_blank">default set of capabilities granted</a> to any container if you don't modify the security context. For example, using *chown* to set file permissions or *net_raw* to craft raw network packages.

Using the pod security context, you can drop default Linux capabilities and/or add non-default Linux capabilities. Again, applying the principle of least-privilege you can greatly reduce the damage of any malicious attack taking over the pod.

As a quick example, you can spawn the <a href="https://github.com/mateobur/kubernetes-securityguide/blob/master/capabilities/flask-cap.yaml" target="_blank">flask-cap pod</a>:

    $ kubectl create -f flask-cap.yaml

    apiVersion: v1
    kind: Pod
    metadata:
      name: flask-cap
      namespace: default
    spec:
      containers:
      - image: mateobur/flask
        name: flask-cap
        securityContext:
          capabilities:
            drop:
              - NET_RAW
              - CHOWN

Note that some *securityContext* should be applied at the pod level, while other labels are applied at container level.

If you spawn a shell, you can verify that these capabilities have been dropped:

    $ kubectl exec -it flask-cap bash
    root@flask-cap:/# ping 8.8.8.8
    ping: Lacking privilege for raw socket.
    root@flask-cap:/# chown daemon /tmp
    chown: changing ownership of '/tmp': Operation not permitted

**AppArmor and Seccomp:** You can also apply the profiles of these security frameworks to Kubernetes pods. This feature is in beta state as of Kubernetes 1.9, <a href="https://raw.githubusercontent.com/kubernetes/website/master/docs/tutorials/clusters/hello-apparmor-pod.yaml" target="_blank">profile configurations are referenced using annotations for the time being</a>.

AppArmor, Seccomp or SELinux allow you to define run-time profiles for your containers, but if you want to define run-time profiles at a higher level with more context, <a href="https://sysdigrp2rs.wpengine.com/opensource/falco/" target="_blank">Sysdig Falco</a> and <a href="https://sysdigrp2rs.wpengine.com/product/secure/" target="_blank">Sysdig Secure</a> can be better options. Sysdig Falco monitors the run-time security of your containers according to a set of user-defined rules, it has some similarities and some important differences with the other tools we just mentioned (reviewed in the "<a href="https://sysdigrp2rs.wpengine.com/blog/selinux-seccomp-falco-technical-discussion/" target="_blank">SELinux, Seccomp, Sysdig Falco, and you</a>" article).

**AllowPrivilegeEscalation:** The <a href="https://www.kernel.org/doc/Documentation/prctl/no_new_privs.txt" target="_blank">execve system call</a> can grant a newly-started program privileges that its parent did not have, such as the *setuid* or *setgid* Linux flags. This is controlled by the AllowPrivilegeEscalation boolean and should be used with care and only when required.

**ReadOnlyRootFilesystem:** This controls whether a container will be able to write into the root filesystem. It is common that the containers only need to write on mounted volumes that persist the state, as their root filesystem is supposed to be immutable. You can enforce this behavior using the *readOnlyRootFilesystem* flag:

    $ kubectl create -f https://raw.githubusercontent.com/mateobur/kubernetes-securityguide/master/readonly/flask-ro.yaml
    $ kubectl exec -it flask-ro bash
    root@flask-ro:/# mount | grep "\/\ "
    none on / type aufs (ro,relatime,si=e6100da9e6227a70,dio,dirperm1)
    root@flask-ro:/# touch foo
    touch: cannot touch 'foo': Read-only file system

## Kubernetes Security Policy {#kubernetespodsecuritypolicy}

Kubernetes Pod Security Policy (PSP), often shortened to Kubernetes Security Policy is implemented as an [admission controller][2]. Using security policies you can restrict the pods that will be allowed to run on your cluster, only if they follow the policy we have defined.

You have different <a href="https://kubernetes.io/docs/concepts/policy/pod-security-policy/" target="_blank">control aspects</a> that the cluster administrator can set:

<table>
  <tr>
    <td>
      <strong>Control Aspect</strong>
    </td>
    
    <td>
      Field Names
    </td>
  </tr>
  
  <tr>
    <td>
      Running of privileged containers
    </td>
    
    <td>
      <a href="https://kubernetes.io/docs/concepts/policy/pod-security-policy/#privileged">privileged</a>
    </td>
  </tr>
  
  <tr>
    <td>
      Usage of the root namespaces
    </td>
    
    <td>
      <a href="https://kubernetes.io/docs/concepts/policy/pod-security-policy/#host-namespaces">hostPID, hostIPC</a>
    </td>
  </tr>
  
  <tr>
    <td>
      Usage of host networking and ports
    </td>
    
    <td>
      <a href="https://kubernetes.io/docs/concepts/policy/pod-security-policy/#host-namespaces">hostNetwork, hostPorts</a>
    </td>
  </tr>
  
  <tr>
    <td>
      Usage of volume types
    </td>
    
    <td>
      <a href="https://kubernetes.io/docs/concepts/policy/pod-security-policy/#volumes-and-file-systems">volumes</a>
    </td>
  </tr>
  
  <tr>
    <td>
      Usage of the host filesystem
    </td>
    
    <td>
      <a href="https://kubernetes.io/docs/concepts/policy/pod-security-policy/#volumes-and-file-systems">allowedHostPaths</a>
    </td>
  </tr>
  
  <tr>
    <td>
      White list of FlexVolume drivers
    </td>
    
    <td>
      <a href="https://kubernetes.io/docs/concepts/policy/pod-security-policy/#flexvolume-drivers">allowedFlexVolumes</a>
    </td>
  </tr>
  
  <tr>
    <td>
      Allocating an FSGroup that owns the pod's volumes
    </td>
    
    <td>
      <a href="https://kubernetes.io/docs/concepts/policy/pod-security-policy/#volumes-and-file-systems">fsGroup</a>
    </td>
  </tr>
  
  <tr>
    <td>
      Requiring the use of a read only root file system
    </td>
    
    <td>
      <a href="https://kubernetes.io/docs/concepts/policy/pod-security-policy/#volumes-and-file-systems">readOnlyRootFilesystem</a>
    </td>
  </tr>
  
  <tr>
    <td>
      The user and group IDs of the container
    </td>
    
    <td>
      <a href="https://kubernetes.io/docs/concepts/policy/pod-security-policy/#users-and-groups">runAsUser, supplementalGroups</a>
    </td>
  </tr>
  
  <tr>
    <td>
      Restricting escalation to root privileges
    </td>
    
    <td>
      <a href="https://kubernetes.io/docs/concepts/policy/pod-security-policy/#privilege-escalation">allowPrivilegeEscalation, defaultAllowPrivilegeEscalation</a>
    </td>
  </tr>
  
  <tr>
    <td>
      Linux capabilities
    </td>
    
    <td>
      <a href="https://kubernetes.io/docs/concepts/policy/pod-security-policy/#capabilities">defaultAddCapabilities, requiredDropCapabilities, allowedCapabilities</a>
    </td>
  </tr>
  
  <tr>
    <td>
      The SELinux context of the container
    </td>
    
    <td>
      <a href="https://kubernetes.io/docs/concepts/policy/pod-security-policy/#selinux">seLinux</a>
    </td>
  </tr>
  
  <tr>
    <td>
      The AppArmor profile used by containers
    </td>
    
    <td>
      <a href="https://kubernetes.io/docs/concepts/policy/pod-security-policy/#apparmor">annotations</a>
    </td>
  </tr>
  
  <tr>
    <td>
      The seccomp profile used by containers
    </td>
    
    <td>
      <a href="https://kubernetes.io/docs/concepts/policy/pod-security-policy/#seccomp">annotations</a>
    </td>
  </tr>
  
  <tr>
    <td>
      The sysctl profile used by containers
    </td>
    
    <td>
      <a href="https://kubernetes.io/docs/concepts/policy/pod-security-policy/#sysctl">annotations</a>
    </td>
  </tr>
</table>

  
  
There is a direct relation between the Kubernetes Pod Security Context labels and the Kubernetes Pod Security Policies. Your Security Policy will filter allowed pod security contexts defining:

*   Default pod security context values (i.e. `defaultAddCapabilities`)
*   Mandatory pod security flags and values (i.e. `allowPrivilegeEscalation: false`)
*   Whitelists and blacklists for the list-based security flags (i.e. list of allowed host paths to mount).

For example, to define that container can only mount an specific host path you would do:

    allowedHostPaths:
      # This allows "/foo", "/foo/", "/foo/bar" etc., but
      # disallows "/fool", "/etc/foo" etc.
      # "/foo/../" is never valid.
      - pathPrefix: "/foo"

You need the *PodSecurityPolicy* [admission controller][2] enabled in your API server to enforce these policies.

If you plan to enable *PodSecurityPolicy*, first configure (or have present already) a default PSP and the <a href="https://kubernetes.io/docs/concepts/policy/pod-security-policy/#authorizing-policies" target="_blank">associated RBAC permissions</a>, otherwise the cluster will fail to create new pods.

If your cloud provider / deployment design already supports and enables PSP, it will come pre-populated with a default set of policies, for example:

    $ kubectl get psp
    NAME                           PRIV      CAPS      SELINUX    RUNASUSER   FSGROUP    SUPGROUP   READONLYROOTFS   VOLUMES
    gce.event-exporter             false     []        RunAsAny   RunAsAny    RunAsAny   RunAsAny   false            [hostPath secret]
    gce.fluentd-gcp                false     []        RunAsAny   RunAsAny    RunAsAny   RunAsAny   false            [configMap hostPath secret]
    gce.persistent-volume-binder   false     []        RunAsAny   RunAsAny    RunAsAny   RunAsAny   false            [nfs secret]
    gce.privileged                 true      [*]       RunAsAny   RunAsAny    RunAsAny   RunAsAny   false            [*]
    gce.unprivileged-addon         false     []        RunAsAny   RunAsAny    RunAsAny   RunAsAny   false            [emptyDir configMap secret]

In the event you enabled PSP for a cluster that didn't have any pre-populated rule, you can create a permissive policy to avoid run-time disruption and then perform iterative adjustments over your configuration:

For example, this policy below will prevent the execution of any pod that tries to use the root user or group, allowing any other security context:

    apiVersion: extensions/v1beta1
    kind: PodSecurityPolicy
    metadata:
      name: example
    spec:
      privileged: true
      seLinux:
        rule: RunAsAny
      supplementalGroups:
        rule: RunAsAny
      runAsUser:
        rule: RunAsAny
      fsGroup:
        rule: 'MustRunAs'
        ranges:
          - min: 1
            max: 65535
      volumes:
      - '*'
    
    $ kubectl create -f psp.yaml
    podsecuritypolicy "example" created
    
    $ kubectl get psp
    NAME                           PRIV      CAPS      SELINUX    RUNASUSER   FSGROUP     SUPGROUP   READONLYROOTFS   VOLUMES
    example                        true      []        RunAsAny   RunAsAny    MustRunAs   RunAsAny   false            [*]

If you try to create new pods without the *runAsUser* directive you will get:

    $ kubectl create -f https://raw.githubusercontent.com/mateobur/kubernetes-securityguide/master/readonly/flask-ro.yaml
    $ kubectl describe pod flask-ro
    ...
    Failed        Error: container has runAsNonRoot and image will run as root

## Kubernetes Network Policies {#kubernetesnetworkpolicies}

Kubernetes also defines security at the <a href="https://kubernetes.io/docs/concepts/services-networking/network-policies/" target="_blank">pod networking level</a>. A network policy is a specification of how groups of pods are allowed to communicate with each other and other network endpoints.

You can compare Kubernetes network policies with classic network firewalling (*ala* iptables) but with one important advantage: using Kubernetes context like pod labels, namespaces, etc.

Kubernetes supports several third-party plugins that implement <a href="https://kubernetes.io/docs/concepts/cluster-administration/network-plugins/" target="_blank">pod overlay networks</a>. You need to check your provider documentation (these for <a href="https://kubernetes.io/docs/tasks/administer-cluster/calico-network-policy/" target="_blank">Calico</a> or <a href="https://kubernetes.io/docs/tasks/administer-cluster/weave-network-policy/" target="_blank">Weave</a>) to make sure that Kubernetes network policies are supported and enabled, otherwise, the configuration will show up in your cluster but will not have any effect.

Let's use the Kubernetes example scenario <a href="https://github.com/kubernetes/kubernetes/tree/master/examples/guestbook" target="_blank">guestbook</a> to show how these network policies work:

    kubectl create -f https://raw.githubusercontent.com/kubernetes/kubernetes/master/examples/guestbook/all-in-one/guestbook-all-in-one.yaml

This will create 'frontend' and 'backend' pods:

    $ kubectl describe pod frontend-685d7ff496-7s6kz | grep tier
            tier=frontend
    $ kubectl describe pod redis-master-7bd4d6ccfd-8dnlq | grep tier
            tier=backend

You can configure your network policy with these logical. Abstracting concepts such as IP address or physical node won't work because Kubernetes can change them dynamically.

Let's apply the following network policy:

    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: deny-backend-egress
      namespace: default
      spec:
        podSelector:
          matchLabels:
            tier: backend
            policyTypes:
              - Egress
              egress:
                - to:
                  - podSelector:
                    matchLabels:
                      tier: backend

That you can also find in the repository:

    $ kubectl create -f netpol/guestbook-network-policy.yaml

Then you can get the pod names and local IP addresses using:

    $ kubectl get pods -o wide
    [...]

In order to check that the policy is working as expected, you can 'exec' into the 'redis-master' pod and try to ping first a 'redis-slave' (same tier) and then a 'frontend' pod:

    $ kubectl exec -it redis-master-7bd4d6ccfd-8dnlq bash
    $ ping 10.28.4.21
    PING 10.28.4.21 (10.28.4.21) 56(84) bytes of data.
    64 bytes from 10.28.4.21: icmp_seq=1 ttl=63 time=0.092 ms
    $ ping 10.28.4.23
    PING 10.28.4.23 (10.28.4.23) 56(84) bytes of data.
    (no response, blocked)

Note that this policy will be enforced even if the pods migrate to another node or they are scaled up/down.

## Kubernetes resource allocation management {#kubernetesresourceallocationmanagement}

Resource limits are usually established to avoid unintended saturation due to design limitations or software bugs, but can also protect against malicious resource abuse. Unauthorized resource consumption that tries to remain undetected is becoming much more common due to <a href="https://sysdigrp2rs.wpengine.com/blog/detecting-cryptojacking/" target="_blank">cryptojacking</a> attempts.

There are two basic concepts: **requests** and **limits**.

**Requests**: The Kubernetes node will check if it has enough resources left to fully satisfy the request before scheduling the pod. Kubernetes makes sure that the actual resource consumption never goes over the configured limits.

You can run a quick example from the `resources/flask-resources.yaml`repository file

    apiVersion: v1
    kind: Pod
    metadata:
      name: flask-resources
      namespace: default
    spec:
      containers:
      - image: mateobur/flask
        name: flask-resources
        resources:
          requests:
            memory: 512Mi
          limits:
            memory: 700Mi
    
    $ kubectl create -f resources/flask-resources.yaml

**Limits**: Are the top resource consumption limit the container can make.

Let's use the <a href="https://people.seas.harvard.edu/~apw/stress/" target="_blank">stress</a> load generator to test the limits:

    root@flask-resources:/# stress --cpu 1 --io 1 --vm 2 --vm-bytes 800M
    stress: info: [79] dispatching hogs: 1 cpu, 1 io, 2 vm, 0 hdd
    stress: FAIL: [79] (416) <-- worker 83 got signal 9

The resources that you can reserve and limit by default using the pod description are:

*   CPU
*   Main memory
*   <a href="https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#local-ephemeral-storage" target="_blank">Local ephemeral storage</a>

There are some third party plugins and cloud providers that can extend the Kubernetes API to allow defining requests and limits over any other kind of logical resources using the <a href="https://kubernetes.io/docs/tasks/configure-pod-container/extended-resource/" target="_blank">Extended Resources</a> interface. You can also configure <a href="https://kubernetes.io/docs/tasks/administer-cluster/quota-memory-cpu-namespace/" target="_blank">resource quotas</a> bounded to a namespace context.

## Next steps {#nextsteps}

This part covered how to configure security at the pod level using Kubernetes orchestration capabilities, as well as manage Kubernetes resource allocation. Now you know how to use Kubernetes Security Context, Kubernetes Security Policy and Kubernetes Network Policy resources to define the container privileges, permissions, capabilities and network communication rules.

Next, we suggest to check out [how to secure Kubernetes etcd, kubelet and the Docker registry][3].

 [1]: https://sysdigrp2rs.wpengine.com/blog/container-security-docker-image-scanning/
 [2]: #admission-controllers
 [3]: https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-kubelet-etcd/