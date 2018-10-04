---
ID: 10390
post_title: 'What&#8217;s new in Kubernetes 1.12?'
author: Jorge Salamero Sanz
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/whats-new-in-kubernetes-1-12/
published: true
post_date: 2018-09-20 07:09:05
---
Here at Sysdig we follow Kubernetes development pretty closely. Next Tuesday the next release of our favourite orchestration tool will get out of the oven freshly baked, so this is a summary of what's new in Kubernetes 1.12!

We have grouped features in things that we find cool, security stuff, storage improvements, cloud providers support and other internal changes. Let's have a look:

## Some cool stuff that we are super exicted about {#somecoolstuffthatwearesuperexictedabout}

### <a href="https://github.com/kubernetes/features/issues/117" target="_blank">#117</a>: Arbitrary / Custom Metrics in the Horizontal Pod Autoscaler (Beta) {#117httpsgithubcomkubernetesfeaturesissues117arbitrarycustommetricsinthehorizontalpodautoscalerbeta}

The new Horizontal Pod Autoscaler specification allows you to assign arbitrary labels to your metrics and use this information to scale.

For example, scaling on `http_requests` but only taking into account the `http_requests` of `method=‘GET’`. The `HorizontalPodAutoscaler` controller can fetch metrics in two different ways: <a href="https://github.com/DirectXMan12/kubernetes.github.io/blob/877c6c224e6d97bbae12b2cb255902c892b08729/docs/user-guide/horizontal-pod-autoscaling/index.md" target="_blank">direct Heapster access, and REST client access</a>. Here at Sysdig we wrote previously about HPAs on: <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-scaler/" target="_blank">How to build a Kubernetes Horizontal Pod Autoscaler using custom metrics</a>.

### <a href="https://github.com/kubernetes/features/issues/21" target="_blank">#21</a>: Pod Vertical Scaling (Beta) {#21httpsgithubcomkubernetesfeaturesissues21podverticalscalingbeta}

Using <a href="https://github.com/kgrygiel/community/blob/master/contributors/design-proposals/vertical-pod-autoscaler.md" target="_blank">Vertical Scaling</a> the resource limits assigned to a pod (or set of pods) can be dynamically determined based on analysis of historical resource utilization, amount of resources available in the cluster and real-time events, such as OOMs. This is particularly useful for pods that are costly to destroy and recreate.

### <a href="https://github.com/kubernetes/features/issues/585" target="_blank">#585</a>: RuntimeClass (Alpha) {#585httpsgithubcomkubernetesfeaturesissues585runtimeclassalpha}

`RuntimeClass` is a new cluster-scoped resource that surfaces container runtime properties to the control plane. RuntimeClasses are assigned to pods through a `runtimeClass` field in the `PodSpec`. This provides a new mechanism for supporting multiple runtimes in a cluster and/or node and select which one to use. Finally Docker and Rkt together :) See more about this again <a href="https://github.com/kubernetes/community/blob/master/keps/sig-node/0014-runtime-class.md" target="_blank">in the design proposal doc</a>.

### <a href="https://github.com/kubernetes/features/issues/382" target="_blank">#382</a>: Taint node by condition (Beta) {#382httpsgithubcomkubernetesfeaturesissues382taintnodebyconditionbeta}

The `Taint node by condition` feature causes the node controller to <a href="https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/#taint-nodes-by-condition" target="_blank">dynamically create taints</a> corresponding to observed node conditions. The user can choose to ignore some of the node’s problems (represented as Node conditions) by adding appropriate pod tolerations. This feature is promoted from alpha to beta in 1.12.

### <a href="https://github.com/kubernetes/features/issues/432" target="_blank">#432</a>: Mount namespace propagation (Stable) {#432httpsgithubcomkubernetesfeaturesissues432mountnamespacepropagationstable}

Mount propagation allows to share volumes mounted by a container to other containers in the same pod, or even to other pods in the same node. It means that any mounts from inside the container are reflected in the host’s mount namespace. An good application is to enable containerization of volume plugins. More info in the <a href="https://kubernetes.io/docs/concepts/storage/volumes/#mount-propagation" target="_blank">volumes documentation</a>.

### <a href="https://github.com/kubernetes/features/issues/495" target="_blank">#495</a>: Configurable Pod process namespace sharing (Beta) {#495httpsgithubcomkubernetesfeaturesissues495configurablepodprocessnamespacesharingbeta}

Users can configure containers within a pod to share a common PID namespace by setting an option in the `PodSpec`. More on this in the <a href="https://kubernetes.io/docs/tasks/configure-pod-container/share-process-namespace/" target="_blank">Kubernetes documentation: share process namespace</a>.

### <a href="https://github.com/kubernetes/features/issues/587" target="_blank">#587</a>: Resource quota API (Beta) {#587httpsgithubcomkubernetesfeaturesissues587resourcequotaapibeta}

The quota system will identify specific resources that are limited by default. With current behavior, resource consumption is unlimited if a quota doesn’t exist. With this feature, consumption will be denied if the quota doesn’t exist, restricting consumption of high-cost resources. More about this in the <a href="https://github.com/kubernetes/website/blob/master/content/en/docs/concepts/policy/resource-quotas.md" target="_blank">resoure quota docs</a>.

[tweet_box design="default" float="none"]This is what's new in #Kubernetes 1.12[/tweet_box] 

## Security {#security}

### <a href="https://github.com/kubernetes/features/issues/43" target="_blank">#43</a>: Kubelet TLS bootstrap (Stable) {#43httpsgithubcomkubernetesfeaturesissues43kubelettlsbootstrapstable}

This is also a featured we covered in our blog post <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-rbac-tls/" target="_blank">Kubernetes Security: RBAC and TLS</a>, now graduating as stable. The kubelet can generate a private key and a signing request (CSR) to sent over to the cluster CA to get the corresponding certificate. You can read more info about it <a href="https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/" target="_blank">here</a>.

### <a href="https://github.com/kubernetes/features/issues/267" target="_blank">#267</a>: Kubelet server TLS certificate rotation (Beta) {#267httpsgithubcomkubernetesfeaturesissues267kubeletservertlscertificaterotationbeta}

The kubelets are able to rotate both its client and/or server certificates, actually we already wrote about this security best practice in our <a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-guide/" target="_blank">Kubernetes Security Guide</a>. We can automatically rotate them through the respective `RotateKubeletClientCertificate` and `RotateKubeletServerCertificate` <a href="https://github.com/kubernetes/website/pull/10232/commits/a4add424885c6380f058e5c49239df7907d38b5c" target="_blank">feature flags in the kubelet that are enabled by default now</a>.

### <a href="https://github.com/kubernetes/features/issues/366" target="_blank">#366</a>: Egress support for Network Policy (Stable) {#366httpsgithubcomkubernetesfeaturesissues366egresssupportfornetworkpolicystable}

NetworkPolicy objects support an `egress` or `to` section to allow or deny traffic based on <a href="https://kubernetes.io/docs/concepts/services-networking/network-policies/" target="_blank">IP ranges or Kubernetes metadata</a> (i.e. `namespace=”proxy”`). Cluster egress mechanisms often require rewriting the source or destination IP of packets. this means that connections from pods to `Service` IPs that get rewritten to cluster-external IPs may or may not be subject to `ipBlock`-based policies (see below).

### <a href="https://github.com/kubernetes/features/issues/367" target="_blank">#367</a>: IPBlock for Network Policies (Stable) {#367httpsgithubcomkubernetesfeaturesissues367ipblockfornetworkpoliciesstable}

NetworkPolicy objects now support CIDR IP blocks to be configured in the rule definitions. You can combine Kubernetes-specific selectors with <a href="https://kubernetes.io/docs/concepts/services-networking/network-policies/" target="_blank">IP-based ones</a> both for ingress and egress policies. For example: “allow connections from any pod in the `‘default’` namespace with the label `‘role=db’` to CIDR `10.0.0.0/24` on TCP port `5978`”.

### <a href="https://github.com/kubernetes/features/issues/460" target="_blank">#460</a>: Encryption at rest KMS integration (Beta) {#460httpsgithubcomkubernetesfeaturesissues460encryptionatrestkmsintegrationbeta}

Data encryption at rest using Google Key Management Service as an encryption provider. You can read more about it on <a href="https://kubernetes.io/docs/tasks/administer-cluster/kms-provider/" target="_blank">KMS providers for data encryption</a>.

## Storage {#storage}

### <a href="https://github.com/kubernetes/features/issues/177" target="_blank">#177</a>: Snapshot / restore volume support for Kubernetes (CRD + external controller) (Alpha) {#177httpsgithubcomkubernetesfeaturesissues177snapshotrestorevolumesupportforkubernetescrdexternalcontrolleralpha}

Similar to how API resources `PersistentVolume` and `PersistentVolumeClaim` are used to provision volumes for users and administrators, `VolumeSnapshotContent` and `VolumeSnapshot` API resources can be provided to create volume snapshots for users and administrators. Read more <a href="https://github.com/xing-yang/website/blob/f53fe7ed8bf0ee98a1c45eb67bc505e309fdce0e/content/en/docs/concepts/storage/volume-snapshots.md" target="_blank">about volume snapshots here</a>.

### <a href="https://github.com/kubernetes/features/issues/561" target="_blank">#561</a>: Topology aware dynamic provisioning (Beta) {#561httpsgithubcomkubernetesfeaturesissues561topologyawaredynamicprovisioningbeta}

Topology aware dynamic provisioning. We will be using this to allow a Pod to request one or more Persistent Volumes (PV) with topology that are compatible with the Pod's other scheduling constraints, such as resource requirements and affinity/anti-affinity policies. Read more here about <a href="https://github.com/kubernetes/website/blob/release-1.12/content/en/docs/concepts/storage/storage-classes.md#allowed-topologies" target="_blank">Allowed topologies</a>.

### <a href="https://github.com/kubernetes/features/issues/557" target="_blank">#557</a>: Kubernetes CSI topology support (Beta) {#557httpsgithubcomkubernetesfeaturesissues557kubernetescsitopologysupportbeta}

When we are using multi-zone clusters, pods can be spread across zones in a specific region, so this means that single-zone storage backends should be provisioned in each zone. The volume binding mode handles when volumen binding and dynamic provisioning should happen.

You can read more about this and its support for different cloud providers <a href="https://github.com/kubernetes/website/blob/release-1.12/content/en/docs/concepts/storage/storage-classes.md#volume-binding-mode" target="_blank">here</a>.

### <a href="https://github.com/kubernetes/features/issues/554" target="_blank">#554</a>: Dynamic maximum volume count (Beta) {#554httpsgithubcomkubernetesfeaturesissues554dynamicmaximumvolumecountbeta}

When dynamic volume limits feature is enabled, Kubernetes automatically determines the node type and supports the appropriate number of attachable volumes for the node and vendor.

You can read more about <a href="https://kubernetes.io/docs/concepts/storage/storage-limits/#dynamic-volume-limits" target="_blank">dynamic volume limits</a> in the Kubernetes documentation.

Some other storage changes made it to this release including <a href="https://github.com/kubernetes/features/issues/594" target="_blank">#594</a> and <a href="https://github.com/kubernetes/features/issues/603" target="_blank">#603</a>.

## Cloud providers support improvements {#cloudproviderssupportimprovements}

### <a href="https://github.com/kubernetes/features/issues/586" target="_blank">#586</a>: Azure Availability Zones (Alpha) {#586httpsgithubcomkubernetesfeaturesissues586azureavailabilityzonesalpha}

Kubernetes 1.12 brings support for Azure availability zones. Nodes in within each availability zone will be added with label `failure-domain.beta.kubernetes.io/zone=<region>-<AZ>` and Azure managed disks storage class will be provisioned taking this into account. Read more about it on <a href="https://github.com/kubernetes/cloud-provider-azure/blob/master/docs/using-availability-zones.md" target="_blank">Using Availability Zones</a>.

### <a href="https://github.com/kubernetes/features/issues/513" target="_blank">#513</a>: Azure Virtual Machine Scale Sets (Stable) {#513httpsgithubcomkubernetesfeaturesissues513azurevirtualmachinescalesetsstable}

This feature adds support for Azure Virtual Machine Scale Sets. This is an Azure technology which let you to create and manage a group of identical and load balanced virtual machines.

You can read more about the Azure VMSS in the <a href="https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/overview" target="_blank">Azure Documentation</a>.

### <a href="https://github.com/kubernetes/features/issues/514" target="_blank">#514</a>: Add Azure support to cluster-autoscaler (Stable) {#514httpsgithubcomkubernetesfeaturesissues514addazuresupporttoclusterautoscalerstable}

This feature adds support for Azure Cluster Autoscaler. As resource demands increase, the cluster autoscaler allows your cluster to grow as required. The Cluster Autoscaler does this scaling your agent nodes based on pending pods.

You can read more about the Azure Cluster Autoscaler in the <a href="https://docs.microsoft.com/en-us/azure/aks/autoscaler" target="_blank">Azure Documentation for AKS</a>.

### <a href="https://github.com/kubernetes/features/issues/604" target="_blank">#604</a>: Azure cross resource group nodes (Alpha) {#604httpsgithubcomkubernetesfeaturesissues604azurecrossresourcegroupnodesalpha}

Adds support for cross resource group (RG) nodes and unmanaged (such as on-prem) nodes in Azure cloud provider. The <a href="https://github.com/kubernetes/cloud-provider-azure" target="_blank">Kubernetes Azure cloud-controller-manger docs</a> have all the info about this too.

### <a href="https://github.com/kubernetes/features/issues/558" target="_blank">#558</a>: GCE PD topology support (Beta) {#558httpsgithubcomkubernetesfeaturesissues558gcepdtopologysupportbeta}

Adds support to the Google Compute Engine Persistent Disk topology (rules to describe accessibility of an object with respect to location in a cluster.

In multi-zone clusters, Pods can be spread across zones in a region. Single-zone storage backends should be provisioned in the zones where Pods are scheduled. See Storage Classes: <a href="https://github.com/kubernetes/website/blob/release-1.12/content/en/docs/concepts/storage/storage-classes.md#gce-pd" target="_blank">GCE PD</a> for more details..

### <a href="https://github.com/kubernetes/features/issues/567" target="_blank">#567</a>: AWS EBS topology support (Beta) {#567httpsgithubcomkubernetesfeaturesissues567awsebstopologysupportbeta}

AWS EBS topology support. Adds topology aware dynamic provisioning support for AWS servers, the same as before but now for AWS EBS. See Storage Classes: <a href="https://github.com/kubernetes/website/blob/release-1.12/content/en/docs/concepts/storage/storage-classes.md#aws-ebs" target="_blank">AWS EBS</a> for more details.

## Kubernetes internals {#kubernetesinternals}

### <a href="https://github.com/kubernetes/features/issues/115" target="_blank">#115</a>: Easier installation and upgrades through ComponentConfig (Alpha) {#115httpsgithubcomkubernetesfeaturesissues115easierinstallationandupgradesthroughcomponentconfigalpha}

In earlier Kubernetes versions, modifying the base configuration of the core cluster components (like the kubelet or the scheduler) was a delicate process, that often required live patching and thus, was not easily automatable.

<a href="https://github.com/kubernetes/features/issues/115" target="_blank">ComponentConfig</a> is an ongoing effort to make components configuration more dynamic and directly reachable through the Kubernetes API.

### <a href="https://github.com/kubernetes/features/issues/288" target="_blank">#288</a>: Improve the multi-platform compatibility (Beta) {#288httpsgithubcomkubernetesfeaturesissues288improvethemultiplatformcompatibilitybeta}

Kubernetes aims to support the multiple architectures, including arm, arm64, ppc64le, s390x and windows platforms. Automated <a href="https://github.com/kubernetes/features/issues/288" target="_blank">CI e2e conformance tests</a> have been deployed to ensure compatibility moving forward. A related goal is to support running clusters with <a href="https://github.com/kubernetes/community/blob/master/contributors/design-proposals/multi-platform.md" target="_blank">nodes of mixed architectures</a>, although we are not there yet though.

### <a href="https://github.com/kubernetes/features/issues/612" target="_blank">#612</a>: Quota by priority (Beta) {#612httpsgithubcomkubernetesfeaturesissues612quotabyprioritybeta}

Pods can be created at a specific priority and you can control a pod's consumption of system resources based on a pod's priority, by using the `scopeSelector`. See <a href="https://kubernetes.io/docs/concepts/configuration/pod-priority-preemption/" target="_blank">Pod priority preemption</a> to know more about this.

### <a href="https://github.com/kubernetes/features/issues/548" target="_blank">#548</a>: Schedule DaemonSet Pods by kube-scheduler (Beta) {#548httpsgithubcomkubernetesfeaturesissues548scheduledaemonsetpodsbykubeschedulerbeta}

This feature is not new, but is enabled by default in 1.12. Instead of being scheduled by the DaemonSet controller, are scheduled by the default scheduler. This means that we will see pods and daemonsets created in Pending state and the scheduler will consider pod priority and preemption.

You can read more about <a href="https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#how-daemon-pods-are-scheduled" target="_blank">how daemon pods are scheduled</a> again in the Kubernetes doc.

### <a href="https://github.com/kubernetes/features/issues/576" target="_blank">#576</a>: APIServer DryRun (Alpha) {#576httpsgithubcomkubernetesfeaturesissues576apiserverdryrunalpha}

Add an apiserver "dry-run" parameter so that requests can be validated and "processed" without actually being persisted. The idea is to be able to send requests to modifying endpoints, and see if the request would have succeeded (admission chain, validation, merge conflicts, …) and/or what would have happened without having it actually happen. More information about this <a href="https://github.com/kubernetes/community/blob/master/keps/sig-api-machinery/0015-dry-run.md#kubernetes-dry-run" target="_blank">on this document from sig-api-machinery team</a>.

### <a href="https://github.com/kubernetes/features/issues/578" target="_blank">#578</a>: Server-side printing in kubectl (Stable) {#578httpsgithubcomkubernetesfeaturesissues578serversideprintinginkubectlstable}

`kubectl get` should get columns back from the server, not the client, and be able to handle this type of server response under all use-cases. Currently, the server sends bare information that’s handled by kubectl to print the columns. Now, the server will be the one who sends the headers and fields for each row so kubectl or any other client is able to reproduce the tabular representation. More on <a href="https://github.com/kubernetes/community/blob/master/contributors/design-proposals/api-machinery/server-get.md#specification-of-server-side-get" target="_blank">this feature design proposal doc</a>.

### <a href="https://github.com/kubernetes/features/issues/579" target="_blank">#579</a>: Updated plugin mechanism for kubectl (Beta) {#579httpsgithubcomkubernetesfeaturesissues579updatedpluginmechanismforkubectlbeta}

Kubectl should support extensions adding new commands as well as overriding specific subcommands (at any depth). This proposal introduces the main design for a plugin mechanism in kubectl. The mechanism is a git-style system, that looks for executables on a user's $PATH whose name begins with kubectl-. This allows plugin binaries to override existing command paths and add custom commands and subcommands to kubectl.

For example, the user experience for switching namespaces could go from:

`kubectl config set-context $(kubectl config current-context) --namespace=mynewnamespace` to `kubectl set-ns mynewnamespace`

where `set-ns` would be a user-provided plugin which would call the initial `kubectl config set-context` command and set the namespace flag according to the value provided as the plugin's first parameter. Have a look at the <a href="https://github.com/kubernetes/community/blob/master/keps/sig-cli/0024-kubectl-plugins.md" target="_blank">design proposal</a> if you are curious about this feature.

### <a href="https://github.com/kubernetes/features/issues/591" target="_blank">#591</a>: Horizontal Pod Autoscaler to reach proper size faster (Beta) {#591httpsgithubcomkubernetesfeaturesissues591horizontalpodautoscalertoreachpropersizefasterbeta}

Horizontal Pod Autoscaler algorithm has been improved to reach the desired size faster, full details on how this works <a href="https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/" target="_blank">here</a>.

### <a href="https://github.com/kubernetes/features/issues/592" target="_blank">#592</a>: TTL after finish (Alpha) {#592httpsgithubcomkubernetesfeaturesissues592ttlafterfinishalpha}

This features will clean up finished jobs automatically. These are usually no longer needed in the system. Keeping them around will generate more load in the API server. A TTL can now be configured to have a controller to clean them. Read more in the <a href="https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/" target="_blank">Jobs docs</a>.

### <a href="https://github.com/kubernetes/features/issues/593" target="_blank">#593</a>: Scheduler checks feasibility and scores a subset of all cluster nodes (Alpha) {#593httpsgithubcomkubernetesfeaturesissues593schedulerchecksfeasibilityandscoresasubsetofallclusternodesalpha}

Before Kubernetes 1.12, kube-scheduler used to check the feasibility of all the nodes in a cluster and then scored the feasible ones. Kubernetes 1.12 has a new feature that allows the scheduler to stop looking for more feasible nodes once it finds a certain number of them. This improves the scheduler's performance in large clusters. More about this <a href="https://github.com/kubernetes/website/blob/release-1.12/content/en/docs/concepts/configuration/scheduler-perf-tuning.md" target="_blank">in the scheduler performance tunning docs</a>.

### <a href="https://github.com/kubernetes/features/issues/614" target="_blank">#614</a>: SCTP support for Services, Pod, Endpoint, and NetworkPolicy (Alpha) {#614httpsgithubcomkubernetesfeaturesissues614sctpsupportforservicespodendpointandnetworkpolicyalpha}

SCTP is now supported as additional protocol alongside TCP and UDP in Pod, Service, Endpoint, and NetworkPolicy. See the issue for more info.

Some other internal changes they made it to this release including <a href="https://github.com/kubernetes/features/issues/575" target="_blank">#575</a>, <a href="https://github.com/kubernetes/features/issues/581" target="_blank">#581</a> and <a href="https://github.com/kubernetes/features/issues/595" target="_blank">#595</a>.