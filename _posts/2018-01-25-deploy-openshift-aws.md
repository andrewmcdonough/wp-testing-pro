---
ID: 6040
post_title: How to deploy Openshift on AWS
author: Mateo Burillo
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/deploy-openshift-aws/
published: true
post_date: 2018-01-25 02:25:20
---
In the following tutorial we will show how to quickly boot an OpenShift Origin multinode deployment on Amazon AWS using CloudFormation and Ansible. We found the <a href="https://github.com/aws-quickstart/quickstart-redhat-openshift" target="_blank" rel="noopener">reference architecture</a> had too many additional dependencies like Lambda, Route53, etc and we wanted to build a simple deployment procedure in a similar fashion to <a href="https://github.com/heptio/aws-quickstart" target="_blank" rel="noopener">Heptio’s Kubernetes</a> AWS quick start.

## Introduction {#introduction}

Kubernetes has had an spectacular market penetration in the container space during 2017. And OpenShift is just the icing on the cake. There are a ton of interesting features that both the OSS oriented (Origin) and commercial versions of OpenShift add on top of vanilla Kubernetes like CI/CD workflows, Docker images registry, etc.. you can read more about <a href="https://www.openshift.com/container-platform/kubernetes.html#comparison" target="_blank" rel="noopener">How Does OpenShift Extend Kubernetes</a> in its own homepage.

Here at Sysdig we regularly spin up and destroy test scenarios for demo environments, testing and development with different orchestration tools like Kubernetes, Docker Swarm, DC/OS Mesos and of course, OpenShift too.

For us, the perfect environment requires to hit a complexity sweet spot: we need a few nodes for high-availability scenarios but at the same time we want something that we can frequently spawn in our cloud without worrying too much about complex deployment, resource use / cost, where anyone from our team can access.

We looked at <a href="https://github.com/minishift/minishift" target="_blank" rel="noopener">minishift</a>, wonderful for developers, but would be too simple (single node and intended to be run locally in your laptop) for the case in point.

<a href="https://aws.amazon.com/quickstart/architecture/openshift/" target="_blank" rel="noopener">Red Hat AWS Quick Start</a> is really well designed, but too complex and with many additional dependencies for our purposes.

So we needed something in the middle, that’s why we created our own OpenShift deployment on AWS scripts, and we hope you can reuse some of it!

[tweet_box design="default" float="none"]How to deploy #OpenShift cluster on #AWS using CloudFormation and #Ansible[/tweet_box] 

## Using CloudFormation to create the OpenShift infrastructure on AWS {#usingcloudformationtocreatetheopenshiftinfrastructureonaws}

Manually creating your AWS resources doesn't work for disposable environments, seriously, don't do it, it is:

*   Prone to human error
*   Tedious, repetitive task
*   It's extremely easy to forget to delete some of the cloud entities, littering your cloud account

So we wrote this (really simple) <a href="https://gist.github.com/mateobur/9435b803b912f3e980aacfb0151670b6" target="_blank" rel="noopener">CloudFormation template for OpenShift</a> that will create:

*   1 CentOS-based master node
*   2 CentOS-based worker or minion nodes
*   A separate VPC and subnets to isolate our environment from any other entities (and also making the scenario clean-up easy and safe)
*   Security groups that will open the following public ports:

*   22 SSH for all host
*   8443 for the OpenShift Web console, master node
*   10250 master proxy to node hosts, master node

You can read more about <a href="https://docs.openshift.com/enterprise/3.1/install_config/install/prerequisites.html#prereq-network-access" target="_blank" rel="noopener">OpenShift standard service</a> ports here.

Two improvements could be implemented on this template:

*   Instead of defining two identical nodes / volumes / etc use <a href="https://docs.aws.amazon.com/autoscaling/ec2/userguide/LaunchConfiguration.html" target="_blank" rel="noopener">LaunchConfiguration</a> and Auto Scaling group, so you just define the entities once and can spawn any number of master / worker nodes using parameters. We didn’t want our clusters to grow automatically, so we didn’t proceed with this.
*   Deploying a bastion host to tunnel SSH connections, like the <a href="https://aws.amazon.com/quickstart/architecture/heptio-kubernetes/" target="_blank" rel="noopener">Heptio’s Kubernetes</a> QuickStart.

Once you have <a href="https://docs.aws.amazon.com/AmazonS3/latest/user-guide/upload-objects.html" target="_blank" rel="noopener">uploaded the template file to your S3 bucket</a> and noted down the file link URL, you can launch it using the AWS cli:

    aws cloudformation create-stack \
     --region <deployment_region> \
     --stack-name <stack_name> \
     --template-url "https://<cloudformation_file_url>" \
     --parameters \
       ParameterKey=AvailabilityZone,ParameterValue=<AvailabilityZone> \
       ParameterKey=KeyName,ParameterValue=<infrastructure_aws_key_name> \
     --capabilities=CAPABILITY_IAM

After 10-15 minutes, your deployment should be ready in "CREATE_COMPLETE" state:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/01/AWS_CloudFormation_OpenShift_Sysdig.png" alt="AWS CloudFormation OpenShift Sysdig" width="600" height="180" class="alignleft size-full wp-image-6046" />][1] 

## Installing OpenShift Origin on CentOS {#installingopenshiftoriginoncentos}

Now we are ready to install OpenShift. We decided to install the community version, known as “Origin” as these clusters are intended for non production usage. The project provides a set of <a href="https://github.com/openshift/openshift-ansible" target="_blank" rel="noopener">Ansible playbooks to automate installation: openshift-ansible</a> and we will be using the 3.6 branch in this example.

First, you need to create an Ansible hosts inventory file, the version we used looks like this:

<script src="https://gist.github.com/5c745060a6abcf3f743ec14ca387f119.js"></script> <noscript>
  <pre><code>
File: AnsibleOpenshiftSysdighost
--------------------------------

[OSEv3:children]
masters
etcd
nodes

[OSEv3:vars]
ansible_ssh_user=centos
ansible_sudo=true
ansible_become=true

deployment_type=origin
os_sdn_network_plugin_name=&#39;redhat/openshift-ovs-multitenant&#39;
openshift_install_examples=true

openshift_docker_options=&#39;--selinux-enabled --insecure-registry 172.30.0.0/16&#39;
openshift_master_identity_providers=[{&#39;name&#39;: &#39;htpasswd_auth&#39;, &#39;login&#39;: &#39;true&#39;, &#39;challenge&#39;: &#39;true&#39;, &#39;kind&#39;: &#39;HTPasswdPasswordIdentityProvider&#39;, &#39;filename&#39;: &#39;/etc/openshift/openshift-passwd&#39;}]
openshift_disable_check=disk_availability,docker_storage,memory_availability

[masters]
master-host-name

[etcd]
master-host-name

[nodes]
master-host-name openshift_node_labels="{&#39;region&#39;:&#39;infra&#39;,&#39;zone&#39;:&#39;east&#39;}" openshift_schedulable=true"
worker1-host-name openshift_node_labels="{&#39;region&#39;: &#39;primary&#39;, &#39;zone&#39;: &#39;east&#39;}"
worker2-host-name openshift_node_labels="{&#39;region&#39;: &#39;primary&#39;, &#39;zone&#39;: &#39;east&#39;}"
</code></pre>
</noscript>

Some highlights:

*   The master node is marked as region *infra* and also as *schedulable*, this way we can run some special pods (like the *router* and *Docker registry*) without needing a dedicated infrastructure host.
*   Httpasswd auth is enabled using the file */etc/openshift/openshift-passwd* in order to require authentication to access the OpenShift Web Console.
*   In this playbook we disable several pre-flight checks with the *openshift_disable_check* config key, this way we can use modest AWS instaces (<a href="https://docs.openshift.org/latest/install_config/install/prerequisites.html#install-config-install-prerequisites" target="_blank" rel="noopener">default requisites</a> are actually pretty high for non-production purposes).

Reading the <a href="https://docs.openshift.org/latest/install_config/install/advanced_install.htm" target="_blank" rel="noopener">Advanced install</a> documentation you will find a wealth of config keys and parameters that you can use to customize your own deployment.

Before running the *openshift-ansible* playbook, we’ll do some pre-configuration of the base CentOS hosts using Ansible too:

<script src="https://gist.github.com/7c6456695758aaf72c746bb65dcafce2.js"></script> <noscript>
  <pre><code>
File: prepare.yml
-----------------

---
- hosts: nodes
  gather_facts: no
  pre_tasks:
    - name: &#39;install python2&#39;
      raw: sudo yum install -y python

  tasks:

    - name: upgrade packages
      yum: state=latest name={{ item }}
      with_items:
        - docker
        - NetworkManager

    - name: enable network-manager
      shell: systemctl enable NetworkManager && systemctl start NetworkManager

    - name: docker storage conf file
      copy:
        content: "DEVS=/dev/xvdb\nVG=docker-vg\n"
        dest: /etc/sysconfig/docker-storage-setup

    - name: docker-storage-setup
      shell: docker-storage-setup

    - name: enable docker
      shell: systemctl enable docker && systemctl start docker
</code></pre>
</noscript>

As you can see, we install some required software packages, enable the Docker Engine daemon and NetworkManager service, prepare the storage device for Docker, etc.

Let's apply this playbook:

    ansible-playbook prepare.yml -i <your_hostfile> --key-file <your_infra_ssh_key>

And now, let's go to the essential part, deploying OpenShift.

First we clone the *openshift-ansible* repository, switching to tag *release-3.6*:

    git clone https://github.com/openshift/openshift-ansible.git
    cd openshift-ansible
    git checkout origin/release-3.6

And then we apply the OpenShift installation playbook over the nodes we configured in the inventory file:

    ansible-playbook -c paramiko -i <your_hostfile> openshift-ansible/playbooks/byo/config.yml --key-file <your_infrastructure_ssh_key>

Be warned that it may take a long time to complete (up to ~hour and a half to finish), but no worries, we are almost done!

## OpenShift cluster post installation configuration {#openshiftclusterpostinstallationconfiguration}

Once the installation playbook is finally completed, you can directly SSH in the host you designated as master and check your cluster's status.

    [centos@ip-10-0-0-7 ~]$ oc get nodes
    NAME                       STATUS    AGE       VERSION
    ip-10-0-0-4.ec2.internal   Ready     11d       v1.6.1+5115d708d7
    ip-10-0-0-7.ec2.internal   Ready     11d       v1.6.1+5115d708d7
    ip-10-0-0-9.ec2.internal   Ready     11d       v1.6.1+5115d708d7

And create a user / pass to access OpenShift Web Console:

    sudo htpasswd -b /etc/openshift/openshift-passwd admin <your_pass>

If you browse to *https://public-master-dns-name:8443* (you will find it on your hosts inventory file and your will receive a self-signed certificate security warning), using the previous credentials you should be able to login into the OpenShift Web Console.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/01/OpenShift_Webconsole_Ansible_Sysdig.png" alt="OpenShift Webconsole Ansible Sysdig" width="700" height="500" class="alignleft size-full wp-image-6053" />][2] 

## Monitoring and secure OpenShift cluster with Sysdig Container Intelligence Platform {#monitoringandsecureopenshiftclusterwithsysdigcontainerintelligenceplatform}

As a bonus point, like in all the cluster deployments we do, we will install the Sysdig agent following the <a href="https://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/211421063-Sysdig-Install-OpenShift" target="_blank" rel="noopener">OpenShift specific instructions</a>. This agent connect the OpenShift cluster to the Sysdig platform (Sysdig Monitor and Sysdig Secure).

First create a project, an account and give it the required access level:

    oc new-project sysdigcloud
    oc create serviceaccount sysdigcloud
    oc adm policy add-cluster-role-to-user cluster-reader system:serviceaccount:sysdigcloud:sysdigcloud
    oc adm policy add-scc-to-user privileged system:serviceaccount:sysdigcloud:sysdigcloud

And then, using the <a href="https://github.com/draios/sysdig-cloud-scripts/blob/master/agent_deploy/kubernetes/sysdig-daemonset.yaml" target="_blank" rel="noopener">Sysdig daemonSet</a> to deploy the agent in each of the cluster nodes:

    oc create -f sysdigcloud_daemonset.yaml​

As detailed in the support documentation, you will need to configure at least the *serviceAccount* and Sysdig Cloud access key in this *yaml* file for it to deploy correctly on OpenShift.

After a few seconds, you will be able to see exactly one Sysdig agent per node:

    $ oc get pods -o wide
    NAME                 READY     STATUS    RESTARTS   AGE       IP         NODE
    sysdig-agent-pdxrw   1/1       Running   0          1d        10.0.0.9   ip-10-0-0-9.ec2.internal
    sysdig-agent-pzwdf   1/1       Running   0          1d        10.0.0.7   ip-10-0-0-7.ec2.internal
    sysdig-agent-rzpvj   1/1       Running   0          1d        10.0.0.4   ip-10-0-0-4.ec2.internal

You can now go to your Sysdig Container Intelligence Platform to start monitoring...

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/01/OpenShift_Sysdig_Monitor_Dashboard.png" alt="OpenShift Sysdig Monitor Dashboard" width="800" height="400" class="alignleft size-full wp-image-6055" />][3] 

...and implementing security policies for your new OpenShift environment!

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/01/OpenShift_Sysdig_Secure_Dashboard-1.png" alt="OpenShift Sysdig Secure Dashboard" width="1966" height="935" class="alignleft size-full wp-image-6064" />][4] 

Hopefully you find this deployment method useful and you can reuse and customize some of the code snippets we used to get started with this OpenShift cluster.

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/01/AWS_CloudFormation_OpenShift_Sysdig.png
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/01/OpenShift_Webconsole_Ansible_Sysdig.png
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/01/OpenShift_Sysdig_Monitor_Dashboard.png
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/01/OpenShift_Sysdig_Secure_Dashboard-1.png