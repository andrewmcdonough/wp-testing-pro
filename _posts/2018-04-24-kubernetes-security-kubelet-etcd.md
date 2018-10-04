---
ID: 7114
post_title: 'Securing Kubernetes components: kubelet, Kubernetes etcd and Docker registry &#8211; Kubernetes security guide (part 3).'
author: Mateo Burillo
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-kubelet-etcd/
published: true
post_date: 2018-04-24 09:23:37
---
In addition of configuring the Kubernetes security features, a fundamental part of Kubernetes security is securing sensitive Kubernetes components such as kubelet and internal Kubernetes etcd. We shouldn't forget either common external resources like the Docker registry we pull images from. In this part, we will learn best practices on how to secure the Kubernetes kubelet, the Kubernetes etcd cluster and how to configure a trusted Docker registry.

The 1st part of this <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-guide/" rel="noopener" target="_blank">Kubernetes Security guide</a> focus on <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-rbac-tls/" rel="noopener" target="_blank">Kubernetes RBAC and TLS certificates</a> while part 2 goes on <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-psp-network-policy/" rel="noopener" target="_blank">Kubernetes Security Context, Kubernetes Security Policy and Kubernetes Network Policy</a>.

[tweet_box design="default" float="none"]Best practices and #howto implement #security on #Kubernetes cluster components: kubelet, etcd and registry[/tweet_box] 

## Kubelet security {#kubeletsecurity}

The <a href="https://kubernetes.io/docs/reference/generated/kubelet/" target="_blank" rel="noopener">kubelet</a> is a fundamental piece of any Kubernetes deployment. It is often described as the "Kubernetes agent" software, and is responsible for implementing the interface between the nodes and the cluster logic.

The main task of a kubelet is managing the local container engine (i.e. Docker) and ensuring that the pods described in the API are defined, created, run and remain healthy ; and then destroyed when appropriate.

There are two different communication interfaces to be considered:

*   Access to the Kubelet REST API from users or software (typically just the Kubernetes API entity)
*   Kubelet binary accessing the local Kubernetes node and Docker engine

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/04/kubelet-api-comms-1.png" alt="" width="962" height="122" class="alignnone size-full wp-image-9733" />  


These two interfaces are secured by default using:

*   Security related configuration (parameters) passed to the kubelet binary - [Next section (Kubelet security - access to the kubelet API)][1].
*   NodeRestriction admission controller - See below [Kubelet security - kubelet access to Kubernetes API][2].
*   RBAC to access the kubelet API resources - See below [RBAC example, accessing the kubelet API with curl][3].

### Kubelet security - access to the kubelet API {#kubeletsecurityaccesstothekubeletapi}

The kubelet security configuration parameters are usually passed as <a href="https://kubernetes.io/docs/reference/generated/kubelet/" target="_blank" rel="noopener">arguments</a> to the binary exec. For newer Kubernetes versions (1.10+) you can also use a <a href="https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/" target="_blank" rel="noopener">kubelet configuration file</a>. Either way, the parameters syntax remain the same.

Let's use this example configuration as reference:

/home/kubernetes/bin/kubelet --v=2 --kube-reserved=cpu=70m,memory=1736Mi --allow-privileged=true --cgroup-root=/ --pod-manifest-path=/etc/kubernetes/manifests --experimental-mounter-path=/home/kubernetes/containerized_mounter/mounter --experimental-check-node-capabilities-before-mount=true --cert-dir=/var/lib/kubelet/pki/ --enable-debugging-handlers=true --bootstrap-kubeconfig=/var/lib/kubelet/bootstrap-kubeconfig --kubeconfig=/var/lib/kubelet/kubeconfig **--anonymous-auth=false** **--authorization-mode=Webhook --client-ca-file=/etc/srv/kubernetes/pki/ca-certificates.crt** --cni-bin-dir=/home/kubernetes/bin --network-plugin=cni --non-masquerade-cidr=0.0.0.0/0 --feature-gates=ExperimentalCriticalPodAnnotation=true

Verify the following Kubernetes security settings when configuring kubelet parameters:

*   `anonymous-auth` is set to `false` to disable anonymous access (it will send *401 Unauthorized* responses to unauthenticated requests).
*   kubelet has a `` `--client-ca-file `` flag, providing a CA bundle to verify client certificates.
*   `` `--authorization-mode `` is not set to *AlwaysAllow*, as the more secure *Webhook* mode will delegate authorization decisions to the Kubernetes API server.
*   `--read-only-port` is set to 0 to avoid unauthorized connections to the read-only endpoint (optional).

### Kubelet security - kubelet access to Kubernetes API {#kubeletsecuritykubeletaccesstokubernetesapi}

As we mentioned in the 1st part of this guide, the level of access granted to a kubelet is determined by the <a href="https://kubernetes.io/docs/admin/admission-controllers/#noderestriction" target="_blank" rel="noopener">NodeRestriction</a> <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-psp-network-policy/#kubernetesadmissioncontrollers" target="_blank" rel="noopener">Admission Controller</a> (on RBAC-enabled versions of Kubernetes, stable in 1.8+).

kubelets are bound to the `system:node` Kubernetes <a href="https://kubernetes.io/docs/admin/authorization/rbac/#role-and-clusterrole" target="_blank" rel="noopener">clusterrole</a>.

If *NodeRestriction* is enabled in your API, your kubelets will only be allowed to modify their own Node API object, and only modify Pod API objects that are bound to their node. It's just a <a href="https://kubernetes.io/docs/admin/admission-controllers/#noderestriction" target="_blank" rel="noopener">static restriction for now</a>.

You can check whether you have this admission controller from the Kubernetes nodes executing the apiserver binary:

    $ ps aux | grep apiserver | grep admission-control
    --admission-control=Initializers,NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,DefaultStorageClass,DefaultTolerationSeconds,NodeRestriction,ResourceQuota

### RBAC example, accessing the kubelet API with curl {#rbacexampleaccessingthekubeletapiwithcurl}

Typically, just the Kubernetes API server will need to use the kubelet REST API. As we mentioned before, this interface needs to be protected as you can <a href="https://medium.com/handy-tech/analysis-of-a-kubernetes-hack-backdooring-through-kubelet-823be5c3d67c" target="_blank" rel="noopener">execute arbitrary pods and exec commands on the hosting node</a>.

You can try to communicate directly with the kubelet API from the node shell:

    # curl  -k https://localhost:10250/pods
    Forbidden (user=system:anonymous, verb=get, resource=nodes, subresource=proxy)

Kubelet uses <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-rbac-tls/" target="_blank" rel="noopener">RBAC for authorization</a> and it's telling you that the default anonymous system account is not allowed to connect.

You need to impersonate the API server kubelet client to contact the secure port:

    # curl --cacert /etc/kubernetes/pki/ca.crt --key /etc/kubernetes/pki/apiserver-kubelet-client.key --cert /etc/kubernetes/pki/apiserver-kubelet-client.crt -k https://localhost:10250/pods | jq .
    
    {
      "kind": "PodList",
      "apiVersion": "v1",
      "metadata": {},
      "items": [
        {
          "metadata": {
            "name": "kube-controller-manager-kubenode",
            "namespace": "kube-system",
    ...

Your port numbers may vary depending on your specific deployment method and initial configuration.

## Securing Kubernetes etcd {#securingkubernetesetcd}

etcd is a key-value distributed database that persists Kubernetes state. The <a href="https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/" target="_blank" rel="noopener">etcd configuration and upgrading guide</a> stresses the security relevance of this component:

"Access to etcd is equivalent to root permission in the cluster so ideally, only the API server should have access to it. Considering the sensitivity of the data, it is recommended to grant permission to only those nodes that require access to etcd clusters."

You can enforce these restrictions in three different (complementary) ways:

*   Regular Linux firewalling (iptables/netfilter, etc).
*   Run-time access protection.
*   PKI-based authentication + parameters to use the configured certs.

### Run-time access protection {#runtimeaccessprotection}

An example of run-time access protection could be making sure that the *etcd* binary only reads and writes from a set of configured directories or network sockets, any run-time access that is not explicitly whitelisted will raise an alarm.

Using <a href="https://sysdigrp2rs.wpengine.com/opensource/falco/" target="_blank" rel="noopener">Sysdig Falco</a> it will look similar to this:

    - macro: etcd_write_allowed_directories
      condition: evt.arg[1] startswith /var/lib/etcd
    
    - rule: Write to non write allowed dir (etcd)
      desc: attempt to write to directories that should be immutable
      condition: open_write and not etcd_write_allowed_directories
      output: "Writing to non write allowed dir (user=%user.name command=%proc.cmdline file=%fd.name)"
      priority: ERROR

See <a href="https://sysdigrp2rs.wpengine.com/blog/runtime-security-kubernetes-sysdig-falco/" target="_blank" rel="noopener">Run-time security behavior monitoring: Kubernetes security policies and audit with Sysdig Falco open-source</a> and <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-harden-kube-system/" target="_blank" rel="noopener">How to harden internal kube-system services</a> to continue reading about Kubernetes run-time security.

### PKI-based authentication for etcd {#pkibasedauthenticationforetcd}

Ideally, you should create two sets of certificate and key pairs to be used exclusively for etcd. One pair will verify member to member connections and the other pair will verify Kubernetes API to etcd connections.

Conveniently, the etcd project provides <a href="https://github.com/coreos/etcd/tree/master/hack/tls-setup" target="_blank" rel="noopener">these scripts</a> to help you generate the certificates.

Once you have all the security artifacts (certificates, keys and authorities), you can secure etcd communications using the following configuration flags:

### etcd peer-to-peer TLS {#etcdpeertopeertls}

This will configure authentication and encryption between etcd nodes. To configure etcd with secure peer to peer communication, use the flags:

*   --peer-key-file=<peer.key>
*   --peer-cert-file=<peer.cert>
*   --peer-client-cert-auth
*   --peer-trusted-ca-file=<etcd-ca.cert>

### Kubernetes API to etcd cluster TLS {#kubernetesapitoetcdclustertls}

To allow Kubernetes API to communicate with etcd, you will need:

*   etcd server parameters:
    *   --cert-file=<path>
    *   --key-file=<path>
    *   --client-cert-auth
    *   --trusted-ca-file=<path> (can be the same you used for peer to peer)
*   Kubernetes API server parameters:
    *   --etcd-certfile=k8sclient.cert
    *   --etcd-keyfile=k8sclient.key

It may seem like a lot of parameters at first sight, but it's just a regular PKI design.

## Using a trusted Docker registry {#usingatrusteddockerregistry}

If you don't specify otherwise, Kubernetes will just pull the Docker images from the public registry Docker Hub. This is fine for testing or learning environments, but not convenient for production, as you probably want to keep images and its content private within your organization.

Allowing users to pull images from a public registry is essentially giving access inside your Kubernetes cluster to any random software found on the Internet. Most of the popular Docker image publishers curate and secure their software, however you don't have any guarantee that your developers are going to pull from trusted authors only.

Providing a trusted repository using cloud services (Docker Hub subscription, Quay.io, Google/AWS/Azure also provide their own service) or locally rolling your own (Docker registry, <a href="http://port.us.org/" target="_blank" rel="noopener">Portus</a> or <a href="https://vmware.github.io/harbor/" target="_blank" rel="noopener">Harbor</a>, etc), are two ways to solve this problem.

You will pre-validate and update every image in your registry. Appart from any QA and testing pipeline you regularly apply to your software, this usually means scanning your Docker images for known vulnerabilities and bad security practices.

Assuming you already have a pre-populated trusted repository, you need to tell Kubernetes how to pull from it and ideally, forbid any other unregistered images.

### Configure private Docker registry in Kubernetes {#configureprivatedockerregistryinkubernetes}

Kubernetes provides a <a href="https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/" target="_blank" rel="noopener">convenient way</a> to configure a private Docker registry and store access credentials, including server URL, as a secret:

    kubectl create secret docker-registry regcred --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>

This data will be base64 encoded and included inline as a field of the new secret:

    {
        "apiVersion": "v1",
        "data": {
            ".dockercfg": "eyJyZWdpc3RyeS5sb2NhbCI6eyJ1c2VybmFtZSI6ImpvaG5kb3ciLCJwYXNzd29yZCI6InNlY3JldHBhc3N3b3JkIiwiZW1haWwiOiJqb2huQGRvZSIsImF1dGgiOiJhbTlvYm1SdmR6cHpaV055WlhSd1lYTnpkMjl5WkE9PSJ9fQ=="
        },
        "kind": "Secret",
        "metadata": {
            "creationTimestamp": "2018-04-08T19:13:52Z",
            "name": "regcred",
            "namespace": "default",
            "resourceVersion": "1752908",
            "selfLink": "/api/v1/namespaces/default/secrets/regcred",
            "uid": "f9d91963-3b60-11e8-96b4-42010a800095"
        },
        "type": "kubernetes.io/dockercfg"
    }

Then, you just need to import this secret using the label *imagePullSecrets* in the pod definition:

    spec:
      containers:
      - name: private-reg-container
        image: <your-private-image>
      imagePullSecrets:
      - name: regcred

You can also associate a <a href="https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#add-imagepullsecrets-to-a-service-account" target="_blank" rel="noopener">serviceAccount with imagePullSecrets</a>, the deployments / pods using such serviceAccount will have access to the secret containing registry credentials.

### Kubernetes trusted image collections: banning non trusted registry {#kubernetestrustedimagecollectionsbanningnontrustedregistry}

Once you have created your trusted image repository and Kubernetes pod deployments are pulling from it, the next security measure is to forbid pulling from any non-trusted source.

There are several, complementary ways to achieve this. You can, for example, use <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-psp-network-policy/" target="_blank" rel="noopener">ValidatingAdmissionWebhooks</a>. This way, the Kubernetes control plane will delegate image validation to an external entity.

You have an example implementation <a href="https://github.com/kelseyhightower/grafeas-tutorial" target="_blank" rel="noopener">here, using Grafeas</a> to only allow container images signed by a specific key, configurable via a configmap.

Using <a href="https://sysdigrp2rs.wpengine.com/product/secure/" target="_blank" rel="noopener">Sysdig Secure</a>, you can also create an image whitelist based on image sha256 hash codes. Any non-whitelisted image will fire an alarm and container execution will be immediately stopped.

<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/04/security_image_whitelist-1.png" alt="" width="1070" height="750" class="alignnone size-full wp-image-9734" />  


## Next steps {#nextsteps}

This post provided some insights on how to secure sensitive Kubernetes components and common external resources such as the Docker registry. Now you know how to secure Kubernetes kubelet and the internal Kubernetes etcd cluster, which are the most important pieces.

Want to dig deeper <a href="https://sysdigrp2rs.wpengine.com/blog/runtime-security-kubernetes-sysdig-falco/" target="_blank" rel="noopener">into Docker runtime security</a>? Next chapters will offer plenty of practical examples and use case scenarios covering Kubernetes runtime threat detection.

Ping us at <a href="https://twitter.com/sysdig" target="_blank" rel="noopener">@sysdig</a> or on our <a href="https://slack.sysdigrp2rs.wpengine.com/" target="_blank" rel="noopener">open source Sysdig Slack group</a> to share anything you feel should be included in this comprehensive <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-guide/" rel="noopener" target="_blank">Kubernetes security guide</a>.

 [1]: #kubeletsecurityaccesstothekubeletapi
 [2]: #kubeletsecuritykubeletaccesstokubernetesapi
 [3]: #rbacexampleaccessingthekubeletapiwithcurl