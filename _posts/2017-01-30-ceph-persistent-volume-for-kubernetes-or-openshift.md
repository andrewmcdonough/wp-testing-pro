---
ID: 3663
post_title: >
  Ceph persistent volume for Kubernetes or
  Openshift
author: Jorge Salamero Sanz
post_excerpt: >
  State aware applications like databases
  or file repositories need access to the
  same file system no matter where the
  container they are running on is
  scheduled. Kubernetes and Openshift call
  this persistent volume.
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/ceph-persistent-volume-for-kubernetes-or-openshift/
published: true
post_date: 2017-01-30 08:37:44
---
State aware applications like databases or file repositories need access to the same file system no matter where the container they are running on is scheduled. Kubernetes and Openshift call this persistent volume.

Previously we went through:

**How to Deploy Ceph on AWS (part 1 of 3)** 
*   [Quick Introduction to Ceph and alternatives][1]
*   [How to Deploy Ceph on AWS][2]
*   [Other Ceph deployment strategies for Kubernetes and Openshift][3]

In this second part will learn how to configure Kuberentes or Openshift to use Ceph as persistent volume.

**Ceph Persistent Volume for Kubernetes or Openshift (part 2 of 3)** 
*   [Using Ceph for Kubernetes or Openshift Persistent Volume][4]

And next piece, will see:

**How to Monitor Ceph: the top 5 metrics to watch (part 3 of 3)** 
*   [How to Monitor Ceph with built-in tools][5]
*   [How to Monitor Ceph with Sysdig Monitor][6]
*   [Top 5 metrics to monitor in your Ceph cluster][7]

#### Ceph Persistent Volume for Kubernetes or Openshift {#pv}

We have our storage cluster ready, but how we can use it within our Kubernetes or Openshift cluster for Docker container volumes?

We have 2 options, store volumes as block storage images in Ceph or <a href="https://github.com/kubernetes/kubernetes/tree/master/examples/volumes/cephfs" target="_blank">mounting CephFS inside Kubernetes Pods</a>. We will follow the first approach for flexibility, performance and features like snapshots.

First, we will create a dedicated pool for our images. Make sure you read <a href="http://docs.ceph.com/docs/jewel/rados/operations/pools/" target="_blank">Ceph Cluster operations: Pools</a> if you run this in production to understand how you choose your number of placement groups. From any Ceph node we will run:

<pre># ceph osd pool create test 128 128
pool 'test' created
</pre>

Then we need to create a block device image inside our pool:

<pre># rbd create myvol --size 1G --pool test
# rbd ls -l test
NAME   SIZE PARENT FMT PROT LOCK 
myvol 1024M          2
</pre>

Note: if your Kubernetes cluster nodes run Ubuntu, you will have to disable some features as a workaround for bug <a href="https://bugs.launchpad.net/ubuntu/+source/ceph/+bug/1578484" target="_blank">#1578484</a>:

<pre># rbd feature disable --pool test myvol exclusive-lock object-map fast-diff deep-flatten</pre>

Now let’s move to our Kubernetes cluster nodes and install Ceph common client packages in all of them:

<pre>$ sudo apt install ceph-fs-common ceph-common
or if using Red Hat / Fedora / CentOS:
$ sudo dnf install ceph
</pre>

Next is to copy the keyring to each of the nodes. You can find it in your ansible folder `fetch/{my-cluster-id}/etc/ceph/ceph.client.admin.keyring`.

We are now ready to start deploying our Kubernetes or Openshift entities. First let’s prepare the secret hash from the keyring we have in the ansible folder:

<pre>$ cat fetch/{my-cluster-id}/etc/ceph/ceph.client.admin.keyring | grep key|awk '{print $3}' | base64
QVFDS3pJaFlWdTBwTWhBQXJESmFWQXVOZTc5ZEZieTJ1bDBMSGc9PQo=
</pre>

The secret entity looks like this:

<script src="https://gist.github.com/3b935a5878902ca5ce4d3cc2b0884805.js"></script> <noscript>
  <pre><code> File: ceph-secret.yaml ---------------------- apiVersion: v1 kind: Secret metadata: name: ceph-secret data: key: QVFDS3pJaFlWdTBwTWhBQXJESmFWQXVOZTc5ZEZieTJ1bDBMSGc9PQo= </code></pre>
</noscript>

And we will create it with kubectl:

<pre># kubectl create -f ceph-secret.yaml
secret "ceph-secret" created
</pre>

We will create now a persistent volume:

<script src="https://gist.github.com/3147fe7d859d5ca08d9df6bcccd6bfd6.js"></script> <noscript>
  <pre><code> File: ceph-pv.yaml ------------------ apiVersion: v1 kind: PersistentVolume metadata: name: ceph-pv spec: capacity: storage: 1Gi accessModes: - ReadWriteMany rbd: monitors: - ec2-54-215-184-122.us-west-1.compute.amazonaws.com:6789 - ec2-54-215-203-194.us-west-1.compute.amazonaws.com:6789 - ec2-54-183-152-97.us-west-1.compute.amazonaws.com:6789 pool: test image: myvol user: admin secretRef: name: ceph-secret fsType: ext4 readOnly: false </code></pre>
</noscript>

<pre># kubectl create -f ceph-pv.yaml
persistentvolume "ceph-pv" created
</pre>

And finally the persistent volume claim:

<script src="https://gist.github.com/86c913b5bf48e728e2446c54922d64a4.js"></script> <noscript>
  <pre><code> File: ceph-pv-claim.yaml ------------------------ kind: PersistentVolumeClaim apiVersion: v1 metadata: name: ceph-claim spec: accessModes: - ReadWriteMany resources: requests: storage: 1Gi </code></pre>
</noscript>

<pre># kubectl create -f ceph-pv-claim.yaml 
persistentvolumeclaim "ceph-claim" created
</pre>

We can have a look at the persistent volumes and claims:

<pre># kubectl get pv
NAME      CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS    CLAIM                REASON    AGE
ceph-pv   1Gi        RWX           Retain          Bound     default/ceph-claim             1m
# kubectl get pvc
NAME         STATUS    VOLUME    CAPACITY   ACCESSMODES   AGE
ceph-claim   Bound     ceph-pv   1Gi        RWX           55s
</pre>

Let’s make some use of the persistent volume, creating a MySQL Pod that mounts it on the database path:

<script src="https://gist.github.com/2eb685481c9c2960495ece2a4ab7caf2.js"></script> <noscript>
  <pre><code> File: ceph-mysql-pvc-pod.yaml ----------------------------- apiVersion: v1 kind: Pod metadata: name: ceph-mysql spec: containers: - name: ceph-mysql image: tutum/mysql ports: - name: mysql-db containerPort: 3306 volumeMounts: - name: mysql-pv mountPath: /var/lib/mysql volumes: - name: mysql-pv persistentVolumeClaim: claimName: ceph-claim </code></pre>
</noscript>

<pre># kubectl create -f ceph-mysql-pvc-pod.yaml 
pod "ceph-mysql" created
</pre>

We can check inside the container and see how the Ceph block device is mounted:

<pre># kubectl exec -ti ceph-mysql mount | grep rbd
/dev/rbd0 on /var/lib/mysql type ext4 (rw,relatime,stripe=1024,data=ordered)
</pre>

Now, we will be able to mount this block device as the Pod moves around our Kubernetes or Openshift cluster!

Still eager to learn more? We were at last KubeCon EU and we loved Kubernetes Storage 101, you should definitely check out!

<iframe width="595" height="485" style="border: 1px solid #CCC; border-width: 1px; margin-bottom: 5px; max-width: 100%;" src="//www.slideshare.net/slideshow/embed_code/key/hizq8v5aUM9CNf" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" allowfullscreen="allowfullscreen"> </iframe><div style="margin-bottom: 5px;">
  <strong> <a href="//www.slideshare.net/kubecon/kubecon-eu-2016-kubernetes-storage-101" title="KubeCon EU 2016: Kubernetes Storage 101" target="_blank">KubeCon EU 2016: Kubernetes Storage 101</a> </strong> from <strong><a target="_blank" href="//www.slideshare.net/kubecon">KubeAcademy</a></strong>
</div>

#### Moving into production

You got your Ceph running, checked. You can now schedule containers in your Kubernetes or Openshift cluster using Ceph as persistent storage backend, checked. But next up, before moving to production an step not to be forgotten: monitoring health status and performance of Ceph. Let’s [move on to part 3][8]!

 [1]: /blog/a-ceph-guide-for-kubernetes-and-openshift-users/#intro
 [2]: /blog/a-ceph-guide-for-kubernetes-and-openshift-users/#deploy
 [3]: /blog/a-ceph-guide-for-kubernetes-and-openshift-users/#alternatives
 [4]: #pv
 [5]: /blog/monitor-ceph-top-5-metrics-watch/#builtin
 [6]: /blog/monitor-ceph-top-5-metrics-watch/#sysdig
 [7]: /blog/monitor-ceph-top-5-metrics-watch/#top5metrics
 [8]: /blog/monitor-ceph-top-5-metrics-watch