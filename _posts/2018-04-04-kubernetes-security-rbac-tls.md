---
ID: 6990
post_title: 'Kubernetes RBAC and TLS certificates &#8211; Kubernetes security guide (part 1).'
author: Mateo Burillo
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-rbac-tls/
published: true
post_date: 2018-04-04 09:55:10
---
Kubernetes RBAC security context is a fundamental part of your Kubernetes security best practices, as well as rolling out TLS certificates / PKI authentication for connecting to the Kubernetes API server and between its components. We will learn how to create a user in Kubernetes, set Kubernetes permissions using RBAC and setup/rotate TLS certificates and security tokens.

The 2nd part of this Kubernetes Security guide will focus on <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-psp-network-policy/" rel="noopener" target="_blank">Kubernetes Security Context, Kubernetes Security Policy and Kubernetes Network Policy</a> while part 3 goes on <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-kubelet-etcd/" rel="noopener" target="_blank">Securing Kubernetes components: kubelet, etcd and Docker registry</a>.

## Kubernetes Role-Based Access Control (RBAC) {#kubernetesrolebasedaccesscontrolrbac}

Kubernetes RBAC is essentially an authorization and access control specification where you define the actions (GET, UPDATE, DELETE, etc) that Kubernetes subjects (i.e. human users, software, kubelets) are allowed to perform over Kubernetes entities (i.e. pods, secrets, nodes). [tweet_box design="default" float="none"]How to set #Kubernetes RBAC and TLS certificates step by step with practical #security examples[/tweet_box] 

RBAC uses the "rbac.authorization.k8s.io" API group to drive authorization decisions. Before getting started, it is important to understand the API group building blocks:

*   **<a href="https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/" target="_blank">Namespaces</a>**: Logical segmentation and isolation, or "virtual clusters". Correct use of Kubernetes namespaces is fundamental for security, as you can group together users, roles and resources according to business logic without granting global privileges for the cluster. Typically you use a namespace to group a project, an application, a team or a customer.

*   **Subjects**: The security "actors".
    
    *   Regular users: Humans or other authorized accesses from outside the cluster. Kubernetes delegates the user creation and management to the administrator. In practice, this means that you can "refer" to a user, as we will see on the Kubernetes RBAC examples below, but there is no *User* API object per se.
    *   <a href="https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/" target="_blank">ServiceAccounts</a>: Used to assign permissions to software entities. Kubernetes creates its own default serviceAccounts, and you can create additional ones for your pods/deployments. Any pod run by Kubernetes gets its own privileges through its serviceAccount, applied to all processes run within the containers of that pod.
    *   Groups of users. Kubernetes user groups are not explicitly created, instead, the API can implicitly group users using a common property, like the <a href="https://kubernetes.io/docs/admin/authorization/rbac/#role-binding-examples" target="_blank">prefix of a serviceAccount</a> or the <a href="https://kubernetes.io/docs/admin/authentication/#x509-client-certs" target="_blank">organization field</a> of a user certificate. As with regular Linux permissions, you can assign RBAC privileges to entire groups.
  
*   **Resources**: The entities that will be accessed by the subjects.
    
    *   Resources can refer to a generic entity ("pod", "deployment", etc), subresources such as the logs coming from a pod ("pod/log") or the particular resource name like an Ingress: "ingress-controller-istio", including custom resources your deployment defines.
    *   Resources can also refer to [Pod Security Policies][1] or PSP.
  
*   **<a href="https://kubernetes.io/docs/admin/authorization/rbac/#role-and-clusterrole" target="_blank">Role and ClusterRole</a>**: A set of permissions over a group of resources, think of it as a "security profile", a Role is always confined to a single namespace, a ClusterRole is cluster-scoped.
    
    *   Before designing your security policy, take into account that Kubernetes RBAC permissions are explicitly additive, there are no "deny" rules.
    *   Some resources only make sense at the cluster level (i.e. nodes), you need to create a ClusterRole to control access in this case.
    *   Roles define a list of actions that can be performed over the resources or **verbs**: GET, WATCH, LIST, CREATE, UPDATE, PATCH, DELETE.
  
*   **<a href="https://kubernetes.io/docs/admin/authorization/rbac/#rolebinding-and-clusterrolebinding" target="_blank">RoleBindings and ClusterRoleBindings</a>**: Grants the permissions defined in a Role or ClusterRole to a subject or group of subjects. Again, RoleBindings are bounded to a certain namespace, ClusterRoleBindings are cluster-global.

Let's start looking at these Role and RoleBinding definitions:

    kind: Role
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      namespace: default
      name: pod-reader
    rules:
    - apiGroups: [""] # "" indicates the core API group
      resources: ["pods"]
      verbs: ["get", "watch", "list"]
    
    # This role binding allows "jane" to read pods in the "default" namespace.
    kind: RoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: read-pods
      namespace: default
    subjects:
    - kind: User
      name: jane
      apiGroup: rbac.authorization.k8s.io
    roleRef:
      kind: Role
      name: pod-reader
      apiGroup: rbac.authorization.k8s.io

We define a Role that grants the verbs ["get", "watch", list"] over any pod resource, only in the "default" namespace. Then, we create a RoleBinding that grants the permissions defined in "pod-reader" to the user "jane".

### Kubernetes RBAC security configuration, authentication and authorization {#kubernetesrbacsecurityconfigurationauthenticationandauthorization}

#### RBAC configuration: API server flags {#rbacconfigurationapiserverflags}

Start by making sure your cluster configuration supports RBAC. The location of the configuration file is your kube-apiserver manifest, and this depends on the deployment method but it's usually inside `/etc/kubernetes/manifests` in either the master node(s) or the apiserver pod.

Look for this flag: `--authorization-mode=Node,RBAC`

Node authorization is used by the kubelets, we will discuss kubelet permissions in the 'Securing Kubernetes components' chapter of this guide.

While the API server has plenty of <a href="https://kubernetes.io/docs/reference/generated/kube-apiserver/" target="_blank">flag options</a>, some are best avoided when taking a best practices approach to security:

*   `--insecure-port`: Opens up access to unauthorized, unauthenticated requests, if this parameter is equal to 0, it means no insecure port.
*   `--insecure-bind-address`: Ideally, you should avoid insecure connections altogether, but in case you really need them, you can use this parameter to just bind to localhost. Make sure this parameter is not set, or at least not set to a network-reachable IP address.
*   `--anonymous-auth`: Enables anonymous requests to the secure port of the API server.

#### How to create Kubernetes users and serviceAccounts {#howtocreatekubernetesusersandserviceaccounts}

When it comes to Kubernetes users and permissions, the best approach is one that applies the principle of least privilege, which promotes minimal user profile privileges based on users' job necessities.:

*   Grant the minimum required access privileges for the task that a user or pod need to carry out.
*   Prefer Role and RoleBinding to their cluster counterparts, when possible. Is much easier to control security when is bounded to independent namespaces.
*   Avoid the use of wildcards `["*"]` when defining access to resources or verbs over these resources, be specific.

ServiceAccounts are used to provide an identity to the processes that run in your pods (similar concept to the *sshd* or *www-data* users in a Linux system). If you don't specify a serviceAccount, these pods will be assigned to the default service account of their namespace.

Using default serviceAccounts can be vague and prone to oversights, try to use service-specific service accounts instead. This way you can granularly control the API access that you grant to any software entity inside your cluster.

#### How to create a Kubernetes serviceAccount step by step {#howtocreateakubernetesserviceaccountstepbystep}

Imagine, for example, that your app needs to query the Kubernetes API to retrieve pod information and state changes because you want to notify and send some updates using webhooks.

You just need 'read-only' access to monitor one specific namespace. Using a serviceAccount you can grant these specific privileges (and nothing else) to your software agent.

The default serviceAccount (the one you will get if you don't specify any) is not able to retrieve this information:

    $ kubectl auth can-i list pods -n default --as=system:serviceaccount:default:default
    no

We have created <a href="https://github.com/mateobur/kubernetes-securityguide" target="_blank">an example deployment to showcase</a> this Kubernetes security feature.

Take a look at the *<a href="https://github.com/mateobur/kubernetes-securityguide/blob/master/rbac/flask.yaml" target="_blank">rbac/flask.yaml</a>* file:

    apiVersion: v1
    kind: Namespace
    metadata:
      name: flask
    ---
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: flask-backend
      namespace: flask
    ---
    kind: Role
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: flask-backend-role
      namespace: flask
    rules:
      - apiGroups: [""]
        resources: ["pods"]
        verbs: ["get", "list", "watch"]
    ---
    kind: RoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: flask-backend-role-binding
      namespace: flask
    subjects:
      - kind: ServiceAccount
        name: flask-backend
        namespace: flask
    roleRef:
      kind: Role
      name: flask-backend-role
      apiGroup: rbac.authorization.k8s.io
    ---
    kind: Deployment
    apiVersion: extensions/v1beta1
    metadata:
      name: flask
      namespace: flask
    spec:
      replicas: 2
      template:
        metadata:
          labels:
            app: flask
        spec:
          serviceAccount: flask-backend
          containers:
          - image: mateobur/flask:latest
            name: flask
            ports:
            - containerPort: 5000

This will create a serviceAccount ("flask backend"), a Role that grants some permissions over the other pods in this "flask" namespace, a RoleBinding associating the serviceAccount and the Role, and finally a deployment of pods that will use the serviceAccount:

    $ kubectl create -f flask.yaml

If you query the secrets for the *flask* namespace, you can verify that an API access token was automatically created for your serviceAccount:

    $ kubectl get secrets -n flask
    NAME                        TYPE                                  DATA      AGE
    default-token-fjfgn         kubernetes.io/service-account-token   3         5m
    flask-backend-token-68b6q   kubernetes.io/service-account-token   3         5m

Then, you can check that the permissions are working as you expect with the `kubectl auth`command, that can query access for verbs, subjects and impersonate other accounts:

    $ kubectl auth can-i list pods -n default --as=system:serviceaccount:flask:flask-backend
    no
    
    $ kubectl auth can-i list pods -n flask --as=system:serviceaccount:flask:flask-backend
    yes
    
    $ kubectl auth can-i create pods -n flask --as=system:serviceaccount:flask:flask-backend
    no

To summarize, you will need to configure a serviceAccount and its related Kubernetes RBAC permissions if your software needs to interact with the hosting Kubernetes cluster. Other examples might include the kubelet agents or a <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-scaler/" target="_blank">Kubernetes Horizontal Pod Autoscaler</a>.

#### How to create a Kubernetes user step by step {#howtocreateakubernetesuserstepbystep}

As we mentioned before, Kubernetes users do not have an explicit API object that you can create, list or modify.

Users are bundled as a parameter of a <a href="https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/#context" target="_blank">configuration context</a> that defines the cluster name, (default) namespace and username:

    $ kubectl config get-contexts
    CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
    *         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin

If you look at the current context, you will note that the user has *client-certificate-data* and *client-key-data* attributes (omitted in the output by default for security reasons).

    $ kubectl config view
    ...
    users:
    - name: kubernetes-admin
      user:
        client-certificate-data: REDACTED
        client-key-data: REDACTED

If you have access to the Kubernetes root certification authority, you can generate a new security context that declares a new Kubernetes user.

So, in order to create a new Kubernetes user, let's start creating a new private key:

    $ openssl genrsa -out john.key 2048

Then, you need to create a certificate signing request containing the public key and other subject information:

    $ openssl req -new -key john.key -out john.csr -subj "/CN=john/O=examplegroup"

Note that Kubernetes will use the *Organization* (O=examplegroup) field to determine user group membership for RBAC.

You will sign this CSR using the root Kubernetes CA, found in `/etc/kubernetes/pki` for this example, the file location in your deployment may vary:

    # openssl x509 -req -in john.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out john.crt
    Signature ok
    subject=/CN=john/O=examplegroup
    Getting CA Private Key

You can inspect the new certificate:

    # openssl x509 -in john.crt -text
    Certificate:
        Data:
            Version: 1 (0x0)
            Serial Number: 11309651818125161147 (0x9cf3f46850b372bb)
        Signature Algorithm: sha256WithRSAEncryption
            Issuer: CN=kubernetes
            Validity
                Not Before: Apr  2 20:20:54 2018 GMT
                Not After : May  2 20:20:54 2018 GMT
            Subject: CN=john, O=examplegroup
            Subject Public Key Info:
                Public Key Algorithm: rsaEncryption
                    Public-Key: (2048 bit)

Let's repeat this process for a second user, so we can show how to assign Kubernetes RBAC permissions to a group:

    $ openssl genrsa -out mary.key 2048
    $ openssl req -new -key mary.key -out mary.csr -subj "/CN=mary/O=examplegroup"
    # openssl x509 -req -in mary.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out mary.crt

You can now register the new credentials and config context:

    $ kubectl config set-credentials john --client-certificate=/home/newusers/john.crt --client-key=/home/newusers/john.key
    $ kubectl config set-context john@kubernetes --cluster=kubernetes --user=john
    Context "john@kubernetes" created.
    
    $ kubectl config get-contexts
    CURRENT   NAME                          CLUSTER      AUTHINFO           NAMESPACE
    *         kubernetes-admin@kubernetes   kubernetes   kubernetes-admin
              john@kubernetes               kubernetes   john

If you want this file to be portable between hosts you need to embed the certificates inline. You can do this automatically appending the `--embed-certs=true` parameter to the `kubectl config set-credentials` command.

Let's use this new user / context:

    $ kubectl config use-context john@kubernetes
    $ kubectl get pods
    Error from server (Forbidden): pods is forbidden: User "john" cannot list pods in the namespace "default"

Ok, this is expected because we haven't assigned any RBAC permissions to our "john" user.

Let's go back to our root admin user and create a new clusterrolebinding:

    $ kubectl config use-context kubernetes-admin@kubernetes
    $ kubectl create clusterrolebinding examplegroup-admin-binding --clusterrole=cluster-admin --group=examplegroup
    clusterrolebinding "examplegroup-admin-binding" created
    
    $ kubectl config use-context john@kubernetes
    $ kubectl get pods
    NAME        READY     STATUS    RESTARTS   AGE
    flask-cap   1/1       Running   0          1m

Please note that we have assigned these credentials to the group rather than the user, so the user 'mary' should have exactly the same access privileges.

## Kubernetes TLS certificates rotation and expiration {#kubernetestlscertificatesrotationandexpiration}

Modern Kubernetes deployments and managed cloud Kubernetes providers will properly configure TLS so the communication between the API server and the kubelets, users and pods is already secured. Thus, we will focus only on the maintenance and rotation aspects of these certificates.

Setting a certificate rotation policy from the start will protect you against the usual key mismanagement or leaking that is bound to happen over long periods of time - an occurrence that is often overlooked.

Let’s explore three Kubernetes TLS certificate rotation and expiration scenarios:

*   kubelet TLS certificate rotation & expiration
*   serviceAccount token rotation
*   Kubernetes user cert rotation & expiration

Note that the current TLS implementation in the Kubernetes API has no way to verify a certificate besides checking the origin. Neither CRL (<a href="https://en.wikipedia.org/wiki/Certificate_revocation_list" target="_blank">Certificate Revocation List</a>) nor OCSP (<a href="https://en.wikipedia.org/wiki/Online_Certificate_Status_Protocol" target="_blank">Online Certificate Status Protocol</a>) are implemented. This means that a lost or exposed certificate will be able to authenticate to the API as long as it hasn't expired.

There are a few ways to mitigate the impact:

*   issue (very) short lived certificates to keep the period of potential exposure small
*   remove the permissions in Kubernetes RBAC. You cannot re-use the username until the certificate has expired
*   recreate the certificate authority and issue new certificates to all active users
*   consider OIDC (<a href="https://kubernetes.io/docs/admin/authentication/#openid-connect-tokens" target="_blank">OpenID Connect</a>) as a alternative authentication method

### Kubernetes kubelet TLS certificate rotation {#kuberneteskubelettlscertificaterotation}

The kubelet serves as the bridge between the node operating system and the cluster logic and thus is a critical security component.

By default the kubelet executable will load its certificates from a regular directory that is passed as argument:

    --cert-dir=/var/lib/kubelet/pki/
    
    /var/lib/kubelet/pki# ls
    kubelet-client.crt  kubelet-client.key  kubelet.crt  kubelet.key

You can regenerate the certs manually using the root CA of your cluster, however, starting from Kubernetes 1.8 there is <a href="https://kubernetes.io/docs/tasks/tls/certificate-rotation/#enabling-client-certificate-rotation" target="_blank">an automated approach</a> at your disposal.

You can instruct your kubelets to renew their certificates automatically as the expiration date approaches using the config flags:

*   `--rotate-certificates`

and

*   `--feature-gates=RotateKubeletClientCertificate=true`

By default, the kubelet certificates expire in one year, you can tune this parameter passing the flag `--experimental-cluster-signing-duration` to the <a href="https://kubernetes.io/docs/reference/generated/kube-controller-manager/" target="_blank">kube-controller-manager</a> binary.

### Kubernetes ServiceAccount token rotation {#kubernetesserviceaccounttokenrotation}

Every time you create a serviceAccount, a Kubernetes secret storing its auth token is automatically generated.

    $ kubectl get serviceaccounts
    NAME             SECRETS   AGE
    default          1         26d
    falco-account    1         18d
    sysdig-account   1         12d
    
    $ kubectl get secrets
    NAME                         TYPE                                  DATA      AGE
    default-token-f2lmn          kubernetes.io/service-account-token   3         26d
    falco-account-token-jvgtz    kubernetes.io/service-account-token   3         18d
    sysdig-account-token-9sjgd   kubernetes.io/service-account-token   3         12d

You can request new tokens from the API and replace the old ones:

    $ kubectl delete secret falco-account-token-jvgtz
    $ cat > /tmp/rotate-token.yaml <<EOF
    apiVersion: v1
    kind: Secret
    metadata:
      name: falco-account-token
      annotations:
        kubernetes.io/service-account.name: falco-account
    type: kubernetes.io/service-account-token
    EOF

If you *describe* the new secret, you will be able to see the new token string. Note that existing pods using this serviceAccount will continue using the old (invalid) token, you may want to plan a rolling update over the affected pods to start using the new token.

Updating serviceAccount tokens is not as common as updating user certs, passwords, etc. There is no fully automated way of doing this other than using the Kubernetes API at the moment. Consider whether or not this security artifact rotation makes sense for your use cases.

### Kubernetes user TLS certificate rotation {#kubernetesusertlscertificaterotation}

As we have seen in the 'How to create a Kubernetes user' example, you can assign a certificate to a user, but there is no *User* API object per se.

When you sign the user certificate using Kubernetes root CA, you can assign an expiration date using the `-days` parameter to enforce routinary rotation:

    openssl x509 -req -in john.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -days 365 -out john.crt
    
    $ openssl x509 -in john.crt -text
    Certificate:
        Data:
            Version: 1 (0x0)
            Serial Number: 11309651818125161149 (0x9cf3f46850b372bd)
        Signature Algorithm: sha256WithRSAEncryption
            Issuer: CN=kubernetes
            Validity
                Not Before: Apr  3 10:43:25 2018 GMT
                Not After : Apr  3 10:43:25 2019 GMT

Then you can replace the old user certificate using the *config set-credentials* command:

    $ kubectl config set-credentials john --client-certificate=/home/newusers/john.crt --client-key=/home/newusers/john.key

## Next steps {#nextsteps}

In this part of the Kubernetes Security guide we have learned how to enable and configure Kubernetes RBAC permissions, use credentials for users and services and rotate TLS certificates and tokens, covering the authentication and authorization part of Kubernetes security. Now you are ready to move to the next part in this series, on how to configure 

[Kubernetes Pod Security with Kubernetes Security Context, Kubernetes Security Policy and and Kubernetes Network Policy.][2]. 
**Acknowledgements:** <a href="https://twitter.com/linuxaddicted" rel="noopener" target="_blank">Daniel Kerwin</a> contributed to this article.

 [1]: #kubernetes-pod-security-policies
 [2]: https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-psp-network-policy/