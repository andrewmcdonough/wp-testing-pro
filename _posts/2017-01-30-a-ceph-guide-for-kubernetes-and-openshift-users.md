---
ID: 3661
post_title: >
  A Ceph guide for Kubernetes and
  Openshift users
author: Jorge Salamero Sanz
post_excerpt: 'Ceph is a self-hosted distributed storage system popular among organizations using containers in production. For those looking for a storage solution in their containerized infrastructure, we created this guide to cover: How to Deploy Ceph on AWS, Quick Introduction to Ceph and alternatives, and How to Deploy Ceph on AWS.'
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/a-ceph-guide-for-kubernetes-and-openshift-users/
published: true
post_date: 2017-01-30 08:26:59
---
<a href="https://ceph.com/" target="_blank">Ceph</a> is a self-hosted distributed storage system popular among organizations using containers in production. For those looking for a storage solution in their containerized infrastructure, we created this guide to cover:

**How to Deploy Ceph on AWS (part 1 of 3)** 

*   [Quick Introduction to Ceph and alternatives][1]
*   [How to Deploy Ceph on AWS][2]
*   [Other Ceph deployment strategies for Kubernetes and Openshift][3]

**Ceph Persistent Volume for Kubernetes or Openshift (part 2 of 3)** 

*   [Using Ceph for Kubernetes or Openshift Persistent Volume][4]

**How to Monitor Ceph: the top 5 metrics to watch (part 3 of 3)** 

*   [How to Monitor Ceph with built-in tools][5]
*   [How to Monitor Ceph with Sysdig Monitor][6]
*   [Top 5 metrics to monitor in your Ceph cluster][7]



### How to deploy Ceph on AWS

##### Quick Introduction to Ceph and alternatives {#intro}

Ceph is a distributed storage system designed for high scalability that provides 3 storage methods:

*   **Object storage**: compatible with AWS s3 or Openstack Swift.
*   **Block storage**: similar to what could be a scaled out version of DRBD, like AWS EBS, Google Persistent Disks or Rackspace Cloud Block Storage.
*   Posix compatible **network file system**: similar to NFS but on steroids with multiple current clients, replication, authentication, etc.

Ceph stores data across different storage pools. It uses an algorithm known as CRUSH to calculate which placement group should contain the object and which object storage daemon (OSD) should store the placement group. OSDs are stored in a traditional file system like BRTS, XFS or ext4.

Due to its block storage capabilities, scalability, clustering, replication and flexibility Ceph has started to become popular among Kubernetes and Openshift users. It’s often used as storage backend on Persistent Volumes for Docker containers. We could consider Ceph as a self hosted version of AWS EBS / Google Persistent Disk, while Kubernetes or Openshift is the self run orchestration platform opposed to AWS ECS or Google Container Engine.

#### Ceph basic terminology



*   **OSD**: software that interacts with the logical disks. Typically nodes running the OSD daemon are called OSDs. These handle data storage and replication, recovery, backfilling and rebalancing.
*   **MON**: nodes running the Ceph monitoring software are called MONs. These monitor the cluster state, membership: monitor map and OSD map, placement groups (PG) map and the CRUSH map, providing consensus for distributed decision-making.
*   **MDS**: software that stores the metadata for Ceph distributed filesystem. This is not used for Ceph Block Store, only on Ceph network file system.
*   **RGW**: nodes running the Ceph gateway software are called (RGW, RADOS gateway). This is only used for Ceph Object Store proving the REST interface.
*   **RBD**: Rados Block Device, is the Linux kernel storage layer that stripes disk images into RADOS objects that can be stored across the Ceph cluster. Merged upstream on Linux 2.6.39.
*   **Placement Group**: is an internal and configurable logic strategy to balance the data objects across the different OSDs.
*   **Pools**: logical groups where to store the objects. An analogy with Kubernetes or Openshift could be namespaces.
*   **RADOS**: Reliable Autonomic Distributed Object Storage, the technology on top of which Ceph has been built. It is a self-healing, self-managing system for storage nodes.
*   **CRUSH** algorithm: Controlled Replication Under Scalable Hashing, deterministic and decentralized placement algorithm used by RADOS and Ceph. This is basically the magic that avoids bottlenecks.



#### Why Ceph? What are the alternatives?

Ceph has become popular for being open-source and free to use. It became the most popular OpenStack storage backend, and it is gaining popularity between Kubernetes and Openshift users because can scale up and out using commodity hardware or cloud instances and has a thin provisioning layer.

Alternatives to Ceph in the context of Docker containers are, to mention a few, <a href="https://www.gluster.org/" target="_blank">GlusterFS</a>, NFS, <a href="https://github.com/ClusterHQ/flocker" target="_blank">Flocker</a> (now defunct as a company but still opensource), <a href="https://infinit.sh/" target="_blank">Infinit</a> (now acquired by Docker, will be opensourced this year), iSCSI from multiple vendors or <a href="https://portworx.com/" target="_blank">Portworx</a>.

### How to deploy Ceph on AWS {#deploy}

If you are just getting started and you want to play with Ceph, your first move will probably be to launch a Ceph cluster in AWS. Once the nodes are up and running, the following is valid for any infrastructure.

To draw something as close as possible to production we will start 3 monitor nodes and 3 storage nodes. Will be using Fedora 25 Cloud in `us-west-1` AWS region, if you prefer using Ubuntu 16.04 just use the `ami-539ac933` instead.

Let’s spawn 3 instances for the monitor nodes with the following command:

<pre>aws ec2 run-instances --image-id ami-efa8f88f --instance-type t2.medium --key-name keyname --associate-public-ip-address</pre>

And 3 instances for the storage nodes with:

<pre>aws ec2 run-instances --image-id ami-efa8f88f --instance-type t2.medium --block-device-mappings file://$(pwd)/fedora-ebs-config.json --key-name keyname --associate-public-ip-address</pre>

The `fedora-ebs-config.json` will have something like so we have a dedicated disk for ODS storage:

<pre>[
  {
    "DeviceName": "/dev/xvdb",
    "Ebs": {
    "DeleteOnTermination": true,
    "VolumeType": "gp2",
    "VolumeSize": 30
    }
  }
] 
</pre>

With the infrastructure up and running we have 2 options, we can use either <a href="https://ekuric.wordpress.com/2016/01/01/ceph-storage-cluster-installation-os-fedora-23/" target="_blank">ceph-deploy</a> which is a set of Python scripts to automate the cluster deployment or my favourite, leverage Ansible to deploy and manage the Ceph cluster with <a href="https://github.com/ceph/ceph-ansible" target="_blank">ceph-ansible</a>.

First we will clone the repo:

<pre>$ git clone https://github.com/ceph/ceph-ansible/</pre>

and we will configure our ansible.cfg for this project:

<pre>[defaults]
action_plugins = plugins/actions
roles_path = ./roles

hostfile = hosts

[ssh_connection]
control_path = %(directory)s/%%h-%%p-%%r
</pre>

Now we need to create our hosts with the inventory:

<pre>[all:vars]
ansible_user=fedora # or ubuntu if using ami-539ac933
ansible_become=true
ansible_ssh_private_key_file=~/.ssh/keyname.pem

[mons]
ec2-54-183-211-140.us-west-1.compute.amazonaws.com
ec2-54-183-81-72.us-west-1.compute.amazonaws.com
ec2-52-53-225-165.us-west-1.compute.amazonaws.com

[osds]
ec2-54-183-76-6.us-west-1.compute.amazonaws.com
ec2-54-215-168-100.us-west-1.compute.amazonaws.com
ec2-54-193-120-179.us-west-1.compute.amazonaws.com
</pre>

Next is to enable the `site.yml` and `group_vars` files:

<pre>$ cp site.yml.sample site.yml
$ cp group_vars/all.yml.sample group_vars/all.yml
$ cp group_vars/mons.yml.sample group_vars/mons.yml
$ cp group_vars/osds.yml.sample group_vars/osds.yml
</pre>

Let’s configure a few things, on `group_vars/all.yml`:

<pre>ceph_origin: 'upstream' # whether use distro packages or upstream
ceph_stable: true # stable or dev release
monitor_interface: eth0 # interface to connect to other nodes in the cluster
journal_size: 5120 # OSD journal size in MB
public_network: 172.31.16.0/20 # front network towards the clients, read more on http://docs.ceph.com/docs/master/rados/configuration/network-config-ref/
</pre>

And on `group_vars/osds.yml`:

<pre>devices: # devices to be used by ceph
  - /dev/xvdb
osd_auto_discovery: true # ansible to configure ceph on the previous device automagically
journal_collocation: true # same disk for data and journal
</pre>

Once all this ready, just simply deploy and wait:

<pre>$ ansible-playbook site.yml</pre>

When the Ansible playbook run completes we are ready to go.

### Other Ceph deployment strategies for Kubernetes and Openshift 

Ideally we should be able to deploy Ceph with containers as we like to do with the rest of our services. This is a work in progress effort, but Huamin Chen from Red Hat presented this on <a href="https://events.linuxfoundation.org/sites/events/files/slides/Lessons%20Learned%20Containerizing%20GlusterFS%20and%20Ceph%20with%20Docker%20and%20Kubernetes.pdf" target="_blank">Lessons Learned Containerizing GlusterFS and Ceph with Docker and Kubernetes</a>.

## 

As of today, deploying <a href="https://github.com/ceph/ceph-docker" target="_blank">Ceph in Docker</a> containers can be done as shown in this video:

## <iframe width="560" height="315" src="https://www.youtube.com/embed/FUSTjTBA8f8" frameborder="0" allowfullscreen="allowfullscreen"></iframe> 

And if you were wondering about Kubernetes, this is also possible as <a href="https://github.com/ceph/ceph-docker/tree/master/examples/kubernetes" target="_blank">documented here</a>. The challenges of these containerized approaches versus a more traditional bare-metal approach are dealing with full cluster reboots (MON nodes need persistent storage) and properly scheduling the OSD pods into the nodes with the physical storage devices.

## 

### Up and running

## 

OK, now you’re up and running with some combination of Ceph, Ansible and OpenShift or Kubernetes, on AWS or elsewhere. Next up, we’ll go into details around understanding Ceph performance, health checks, the top 5 Ceph metrics to watch and more. [Keep reading][8]!

 [1]: #intro
 [2]: #deploy
 [3]: #alternatives
 [4]: /blog/ceph-persistent-volume-for-kubernetes-or-openshift/#pv
 [5]: /blog/monitor-ceph-top-5-metrics-watch/#builtin
 [6]: /blog/monitor-ceph-top-5-metrics-watch/#sysdig
 [7]: /blog/monitor-ceph-top-5-metrics-watch/#top5metrics
 [8]: /blog/ceph-persistent-volume-for-kubernetes-or-openshift