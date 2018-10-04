---
ID: 2166
post_title: The Container Ecosystem Project
author: Chris Crane
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/the-container-ecosystem-project/
published: true
post_date: 2015-10-21 06:00:36
---
The ecosystem of awesome new technologies emerging around containers and microservices can be a little overwhelming, to say the least. We thought we might be able to help: welcome to the Container Ecosystem Project. The goals of this project are (1) to clearly lay out the different types technologies that make up the growing container ecosystem and the microservices technology stack - starting from the lowest levels of core container technology, and rising up through layers of abstraction to full-blown container platforms and support tools - and (2) to put forth the latest and greatest examples of each type of technology. This project is a living document - please suggest edits to [the github repo][1] and see below for [more info][2]. 

## Table of contents

*   The Container Ecosystem

*   [Core Container Technologies][3]
*   [Distributed Container Technologies][4]
*   [Container Platform Technologies][5]
*   [Container-Native Support Technologies][6]

*   [About the Container Ecosystem Project][2]
*   [Further Reading][7]
## The Container Ecosystem ([View on Github][1]) 

<div class="table-responsive" id="core">
  <table id="ecosystem" class="table table-bordered">
    <col width="16%"/> <col width="21%"/> <col width="21%"/> <col width="21%"/> <col width="21%"/> <tr>
      <td colspan="5">
        <h3>
          Core Container Technologies
        </h3>
        
        <span style="font-size: 18px">Use these tools to run a small number of containers on a single host</span>
      </td>
    </tr>
    
    <tr style="background-color: #f1f1f2">
      <td>
      </td>
      
      <td style="text-align: center">
        <b> <span>Docker open source</span></b>
      </td>
      
      <td style="text-align: center">
        <b> <span>CoreOS open source<span></span></span></b>
      </td>
      
      <td style="text-align: center">
        <b> <span>Other open source</span></b>
      </td>
      
      <td style="text-align: center">
        <b> <span>Commercial</span></b>
      </td>
    </tr>
    
    <tr>
      <td>
        <span><b>Container specifications</b></span><br /> <span>An abstract definition of a standard “container”, allowing an ecosystem of technologies to support a standard container with potentially multiple, interchangeable runtime implementations</span>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://github.com/appc/spec" target="_blank">Open Container spec</a>: open industry standard for container runtimes; supported by Docker, CoreOS, and most industry leaders; backed by the <a href="http://www.opencontainers.org/" target="_blank">Open Container Initiative (OCI)</a> (run by the Linux Foundation); currently absorbing CoreOS’s AppC standard
          </li>
        </ul>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://github.com/appc/spec" target="_blank">AppC</a> (deprecated): CoreOS is now supporting the OCI
          </li>
        </ul>
      </td>
      
      <td>
      </td>
      
      <td>
      </td>
    </tr>
    
    <tr>
      <td>
        <span><b>Container runtimes</b></span><br /> <span>This is your actual running container (essentially an abstraction of Linux kernel components like namespaces and cgroups that allow virtualization on top of a shared kernel)</span>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://github.com/opencontainers/runc" target="_blank">runc</a>: Docker’s container runtime, now donated to the OCI as the initial implementation of the standard; essentially a repackaging of libcontainer
          </li>
          <li>
            <a href="https://github.com/opencontainers/runc/tree/master/libcontainer" target="_blank">libcontainer</a>: a Linux container library; enables and abstracts interactions with Linux kernel components to create and control containers
          </li>
        </ul>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://github.com/coreos/rkt" target="_blank">rkt</a>: CoreOS’s container runtime; initially an implementation of the AppC specification, which is now being rolled into the OCI spec
          </li>
        </ul>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://linuxcontainers.org/" target="_blank">LXC</a>: a Linux container library; originally utilized by runc until release of libcontainer
          </li>
          <li>
            <a href="https://openvz.org/Main_Page" target="_blank">OpenVZ</a>: a Linux container library
          </li>
        </ul>
      </td>
      
      <td>
      </td>
    </tr>
    
    <tr>
      <td>
        <span><b>Container management</b></span><br /> <span>These tools abstract low level control of your container runtime adding further functionality and usability</span>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://www.docker.com/docker-engine" target="_blank">Docker Engine</a> (aka “Docker”): the core of Docker and its primary interface; creates and runs Docker containers; includes: <ul>
              <li>
                Docker daemon: runs as a process on the host machine and provides an API that abstracts basic container control functions
              </li>
              <li>
                Docker client: a CLI for interacting with the Docker daemon
              </li>
            </ul>
          </li>
        </ul>
      </td>
      
      <td>
        <ul>
          <li>
            <span><a href="https://coreos.com/rkt/docs/latest/commands.html" target="_blank">rkt CLI</a>: rkt’s container management functionality is delivered on-demand by a binary, rather than a daemon background process</span>
          </li>
        </ul>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://linuxcontainers.org/lxd/" target="_blank">LXD</a>: daemon and UI for LXC
          </li>
          <li>
            <a href="http://libvirt.org/" target="_blank">libvirt</a>: container and virtualization mgmt library that supports LXC, OpenVZ, and a variety of hypervisor technologies
          </li>
        </ul>
      </td>
      
      <td>
      </td>
    </tr>
    
    <tr>
      <td>
        <span><b>Container definition</b></span><br /> <span>These tools allow you to define specific containers, so they can be saved, shared and reproduced</span>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://docs.docker.com/userguide/dockerimages/" target="_blank">Docker image</a>: a template representing a fully configured container; Docker container runtimes are created from these images; images are created with Dockerfiles and shared over registries
          </li>
          <li>
            <a href="https://docs.docker.com/articles/dockerfile_best-practices/" target="_blank">Dockerfile</a>: text file containing all the commands needed to build a Docker image
          </li>
        </ul>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://github.com/coreos/rkt#rkt-basics" target="_blank">ACI (App Container Image)</a>: rkt’s native container image format (note, rkt also supports Docker images)
          </li>
        </ul>
      </td>
      
      <td>
      </td>
      
      <td>
      </td>
    </tr>
    
    <tr>
      <td>
        <span><b>Registries</b></span><br /> <span>Repositories for storing and sharing container images</span>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://github.com/docker/distribution" target="_blank">Docker Registry</a>: open source Docker image registry that can be hosted in your own environment
          </li>
        </ul>
      </td>
      
      <td>
      </td>
      
      <td>
      </td>
      
      <td>
        <ul>
          <li>
            Hosted
          </li>
        </ul>
        
        <ul>
          <li>
            <a href="https://aws.amazon.com/ecr/" target="_blank">Amazon EC2 Container Registry (ECR)</a>: still in beta
          </li>
          <li>
            <a href="https://hub.docker.com/" target="_blank">Docker Hub</a>: hosted registry with free paid tiers, private public repositories, and a collection of “official” images
          </li>
          <li>
            <a href="https://cloud.google.com/container-registry/" target="_blank">Google Container Registry</a>
          </li>
          <li>
            <a href="https://quay.io/" target="_blank">Quay.io</a>: CoreOS’s hosted registry
          </li>
        </ul>
        
        <li>
          On-premise
        </li>
        <ul>
          <li>
            <a href="https://www.docker.com/docker-trusted-registry" target="_blank">Docker Trusted Registry</a>
          </li>
          <li>
            <a href="https://coreos.com/products/enterprise-registry" target="_blank">CoreOS Enterprise Registry</a>
          </li>
        </ul>
      </td>
    </tr>
    
    <tr>
      <td>
        <span><b>Operating systems</b></span><br /> <span>OS’s that are designed for hosting containers</span>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="http://boot2docker.io/" target="_blank">boot2docker</a> (basically deprecated by Docker Machine): minimalist Linux for running Docker on PC and Mac in a VM; now used by Docker Machine in certain environments
          </li>
        </ul>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://coreos.com/" target="_blank">CoreOS</a>: minimalist OS built for running distributed, containerized apps; includes etcd and fleet
          </li>
        </ul>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://github.com/rancher/os" target="_blank">RancherOS</a>: minimalist, fully containerized OS
          </li>
          <li>
            <a href="http://www.projectatomic.io/" target="_blank">Project Atomic</a>: minimalist Red Hat Linux; versions include RHEL Atomic, CentOS Atomic, and Fedora Atomic
          </li>
          <li>
            <a href="https://developer.ubuntu.com/en/snappy/" target="_blank">Ubuntu Core "Snappy"</a>: minimalist Ubuntu
          </li>
          <li>
            <a href="https://smartos.org/" target="_blank">SmartOS</a>: Solaris-based OS from Joyent that includes <a href="http://wiki.smartos.org/display/DOC/Zones" target="_blank">Zones</a> (ie. Solaris containers)
          </li>
          <li>
            <a href="https://vmware.github.io/photon/" target="_blank">Photon OS</a>: minimalist OS from VMWare
          </li>
        </ul>
      </td>
      
      <td>
      </td>
    </tr>
    
    <tr>
      <td>
        <span><b>VM management</b></span><br /> <span>These tools help you manage the host virtual environments in which you run your containers</span>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://github.com/docker/machine" target="_blank">Docker Machine</a>: creates and manages host VMs running Docker, including local VMs (eg. VirtualBox) and cloud VMs (eg. Amazon AWS, Google GCP)
          </li>
        </ul>
      </td>
      
      <td>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://www.vagrantup.com/" target="_blank">Hashicorp Vagrant</a>: creates pre-configured VMs for dev environments based on a variety of “Providers” (virtualization technologies) including Docker containers
          </li>
          <li>
            <a href="https://www.ottoproject.io/" target="_blank">Hashicorp Otto</a>: extends Vagrant to deploy and manage VMs across many platforms
          </li>
        </ul>
      </td>
      
      <td>
      </td>
    </tr>
  </table>
</div>

<div class="table-responsive" id="distributed">
  <table id="ecosystem" class="table table-bordered">
    <col width="16%"/> <col width="21%"/> <col width="21%"/> <col width="21%"/> <col width="21%"/> <tr>
      <td colspan="5">
        <h3>
          Distributed Container Technologies
        </h3>
        
        <span style="font-size: 18px">Use these technologies to run applications on a distributed cluster of containers</span>
      </td>
    </tr>
    
    <tr style="background-color: #f1f1f2">
      <td>
      </td>
      
      <td style="text-align: center">
        <b> <span>Docker open source</span></b>
      </td>
      
      <td style="text-align: center">
        <b> <span>CoreOS open source<span></span></span></b>
      </td>
      
      <td style="text-align: center">
        <b> <span>Other open source</span></b>
      </td>
      
      <td style="text-align: center">
        <b> <span>Commercial</span></b>
      </td>
    </tr>
    
    <tr>
      <td>
        <span><b>Scheduling</b></span><br /> <span>These tools manage placement of new containers across abstracted underlying resources</span>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://github.com/docker/swarm/" target="_blank">Docker Swarm</a>: designed to extend Docker API to a cluster; includes scheduling and service discovery
          </li>
        </ul>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://github.com/coreos/fleet" target="_blank">fleet</a>: low level orchestration included in CoreOS; supports basic scheduling; can be used to bootstrap Kubernetes for higher level orchestration
          </li>
        </ul>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://github.com/mesos/chronos" target="_blank">Chronos</a>: framework for scheduling on Mesos
          </li>
        </ul>
      </td>
      
      <td>
      </td>
    </tr>
    
    <tr>
      <td>
        <span><b>Cluster definition</b></span><br /> <span>These tools allow you to define and manage a cluster of dependent containers as a single composable entity</span>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://github.com/docker/docker/issues/9694" target="_blank">Docker Compose</a>: text files used to define and configure a distributed application across a cluster of Docker containers
          </li>
        </ul>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://github.com/coreos/fleet/blob/master/Documentation/unit-files-and-scheduling.md" target="_blank">fleet unit file</a>: fleet uses a specialized version of systemd unit files to define a distributed application across containers
          </li>
        </ul>
      </td>
      
      <td>
      </td>
      
      <td>
      </td>
    </tr>
    
    <tr>
      <td>
        <span><b>Service discovery / Distributed configuration storage</b></span><br /> <span>These tools allow applications within different containers to discover each other and share configuration information (eg. IP addresses or application settings); usually implemented as a globally distributed key-value store</span>
      </td>
      
      <td>
        <ul>
          <li>
            Docker Swarm comes with built in service discovery, but can also use etcd, Consul, Zookeeper
          </li>
        </ul>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://github.com/coreos/etcd" target="_blank">etcd</a>: globally distributed key-value store; included with CoreOS for service discovery
          </li>
        </ul>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://github.com/mesosphere/marathon" target="_blank">Marathon</a>: framework for initializing long running jobs on Mesos; includes service discovery and cluster management functionality
          </li>
          <li>
            <a href="https://www.consul.io/" target="_blank">Hashicorp Consul</a>: service discovery, key/value store, and cluster health checking; uses <a href="https://www.serfdom.io/" target="_blank">Serf</a>
          </li>
          <li>
            <a href="https://zookeeper.apache.org/" target="_blank">Apache ZooKeeper</a>: globally distributed key-value store
          </li>
        </ul>
      </td>
      
      <td>
      </td>
    </tr>
    
    <tr>
      <td>
        <span><b>Dynamic configuration management</b></span><br /> <span>These tools let you dynamically update application settings based on changes to your distributed key-value store in applications that don’t natively support this</span>
      </td>
      
      <td>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="http://www.confd.io/" target="_blank">confd</a>: originally built for etcd, but now supports Consul and ZooKeeper
          </li>
        </ul>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://github.com/hashicorp/consul-template" target="_blank">Consul Template</a>: built natively for Consul
          </li>
        </ul>
      </td>
      
      <td>
      </td>
    </tr>
  </table>
</div>

<div class="table-responsive" id="platform">
  <table id="ecosystem" class="table table-bordered">
    <col width="16%"/> <col width="21%"/> <col width="21%"/> <col width="21%"/> <col width="21%"/> <tr>
      <td colspan="5">
        <h3>
          Container Platform Technologies
        </h3>
        
        <span style="font-size: 18px">Use these technologies as complete platforms for running distributed applications across clusters of containers</span>
      </td>
    </tr>
    
    <tr style="background-color: #f1f1f2">
      <td>
      </td>
      
      <td style="text-align: center">
        <b> <span>Docker open source</span></b>
      </td>
      
      <td style="text-align: center">
        <b> <span>CoreOS open source<span></span></span></b>
      </td>
      
      <td style="text-align: center">
        <b> <span>Other open source</span></b>
      </td>
      
      <td style="text-align: center">
        <b> <span>Commercial</span></b>
      </td>
    </tr>
    
    <tr>
      <td>
        <span><b>Container orchestration platforms</b></span><br /> <span>These platforms include or abstract away all of the core functionality (listed above) needed for container cluster management (“orchestration”), including container management, scheduling, cluster definition, and service discovery</span>
      </td>
      
      <td>
        <ul>
          <li>
            Docker Swarm, Compose, and Machine can all run together to create a complete orchestration platform (still beta); Docker Swarm can also support more advanced orchestration tools like Kubernetes
          </li>
        </ul>
      </td>
      
      <td>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="http://mesos.apache.org/" target="_blank">Apache Mesos</a>: mature, highly scalable service that abstracts a pool of underlying resources and distributes “tasks” (including Docker images) from various application frameworks; uses Marathon and Chronos to add cluster management, scheduling, and service discovery; also can support Kubernetes
          </li>
          <li>
            <a href="http://kubernetes.io/" target="_blank">Kubernetes</a>: orchestration platform designed specifically for running microservices on clusters of containers; includes scheduling, cluster management and service discovery through abstractions such as “pods”, “replication controllers (RCs)”, and “services”; originally from Google, now donated to the <a href="https://cncf.io/" target="_blank">CNCF</a>
          </li>
          <li>
            <a href="https://nomadproject.io/" target="_blank">Hashicorp Nomad</a>: uses Consul
          </li>
        </ul>
      </td>
      
      <td>
      </td>
    </tr>
    
    <tr>
      <td>
        <span><b>Hosted container platforms</b></span><br /> <span>These platforms offer container hosting and orchestration as a service</span>
      </td>
      
      <td>
      </td>
      
      <td>
      </td>
      
      <td>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://aws.amazon.com/ecs/" target="_blank">Amazon EC2 Container Service (ECS)</a>
          </li>
          <li>
            <a href="https://cloud.google.com/container-engine/" target="_blank">Google Container Engine</a>: uses Kubernetes
          </li>
          <li>
            <a href="https://www.tutum.co/" target="_blank">Tutum</a>: still beta
          </li>
          <li>
            <a href="https://enterprise.openshift.com/" target="_blank">Redhat Openshift</a>: uses Kubernetes
          </li>
          <li>
            <a href="https://www.joyent.com/" target="_blank">Joyent - Triton</a>
          </li>
          <li>
            <a href="https://giantswarm.io/" target="_blank">Giant Swarm</a>: still beta
          </li>
          <li>
            <a href="https://www.profitbricks.com/docker" target="_blank">ProfitBricks</a>: still beta
          </li>
          <li>
            <a href="https://modulus.io/" target="_blank">Modulus</a>
          </li>
          <li>
            <a href="https://getcarina.com/" target="_blank">Rackspace Carina</a>: still beta
          </li>
        </ul>
      </td>
    </tr>
    
    <tr>
      <td>
        <span><b>Container platform management</b></span><br /> <span>These technologies add further abstracted management and control layers to distributed container environments, often through GUIs</span>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://youtu.be/BKVKc_xFnw8?list=PLenh213llmcbpJ78mZdh5pnJ_feVT9bezt=5237" target="_blank">Project Orca</a>: opinionated management GUI built on top of full stack of Docker technologies; still alpha
          </li>
        </ul>
      </td>
      
      <td>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://github.com/rancher/rancher" target="_blank">Rancher</a>: still beta
          </li>
          <li>
            <a href="https://github.com/containership/containership" target="_blank">ContainerShip</a>
          </li>
          <li>
            <a href="http://panamax.io/" target="_blank">Panamax</a>
          </li>
          <li>
            <a href="https://github.com/shipyard/shipyard" target="_blank">Shipyard</a>
          </li>
          <li>
            <a href="https://github.com/joyent/sdc" target="_blank">Joyent SmartDataCenter</a>: uses SmartOS
          </li>
        </ul>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://mesosphere.com/" target="_blank">Mesosphere DCOS</a>: uses Mesos
          </li>
          <li>
            <a href="https://tectonic.com/" target="_blank">CoreOS Tectonic</a>: uses CoreOS+Kubernetes; still beta
          </li>
          <li>
            <a href="http://nirmata.com/" target="_blank">Nirmata</a>: multi-cloud container management; built in scheduling, policy-based orchestration, service discovery, dynamic load balancing, and infrastructure optimization
          </li>
          <li>
            <a href="http://containership.io/">ContainerShip Enterprise</a>: still beta
          </li>
          <li>
            <a href="http://stackengine.com/" target="_blank">StackEngine</a>
          </li>
          <li>
            <a href="http://www.appformix.com/" target="_blank">AppFormix</a>
          </li>
        </ul>
      </td>
    </tr>
    
    <tr>
      <td>
        <span><b>Container-based PaaS</b></span><br /> <span>These platforms further abstract container-based infrastructures by managing application code deployment and offering PaaS-like user experiences</span>
      </td>
      
      <td>
      </td>
      
      <td>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://github.com/deis/deis" target="_blank">Deis</a>: container based PaaS; uses CoreOS
          </li>
          <li>
            <a href="https://github.com/flynn/flynn" target="_blank">Flynn</a>: container based PaaS; uses etcd
          </li>
          <li>
            <a href="http://www.openshift.org/" target="_blank">RedHat Openshift Origin</a>
          </li>
          <li>
            <a href="https://github.com/CiscoCloud/microservices-infrastructure" target="_blank">Cisco Mantl</a>: uses Mesos
          </li>
          <li>
            <a href="https://github.com/progrium/dokku" target="_blank" target="_blank">Dokku</a>: minimalist PaaS
          </li>
          <li>
            <a href="https://github.com/remind101/empire" target="_blank">Empire</a>: PaaS built for Amazon's ECS
          </li>
        </ul>
      </td>
      
      <td>
      </td>
    </tr>
  </table>
</div>

<div class="table-responsive" id="support">
  <table id="ecosystem" class="table table-bordered">
    <col width="16%"/> <col width="21%"/> <col width="21%"/> <col width="21%"/> <col width="21%"/> <tr>
      <td colspan="5">
        <h3>
          Container-Native Support Technologies
        </h3>
        
        <span style="font-size: 18px">Use these additional container-native tools to support your container-based infrastructure</span>
      </td>
    </tr>
    
    <tr style="background-color: #f1f1f2">
      <td>
      </td>
      
      <td style="text-align: center">
        <b> <span>Docker open source</span></b>
      </td>
      
      <td style="text-align: center">
        <b> <span>CoreOS open source<span></span></span></b>
      </td>
      
      <td style="text-align: center">
        <b> <span>Other open source</span></b>
      </td>
      
      <td style="text-align: center">
        <b> <span>Commercial</span></b>
      </td>
    </tr>
    
    <tr>
      <td>
        <b>Networking</b>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://docs.docker.com/articles/networking/" target="_blank">Docker port expose</a>: Docker feature that links a container port to a host port
          </li>
          <li>
            <a href="https://docs.docker.com/userguide/dockerlinks/" target="_blank">Docker linking</a>: Docker feature offering a basic connection between containers on the same host
          </li>
          <li>
            <a href="https://github.com/docker/libnetwork" target="_blank">libnetwork</a>: advanced container networking library (still “under heavy development”)
          </li>
        </ul>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://github.com/coreos/flannel" target="_blank">flannel</a>: overlay network built using etcd that gives each host a separate subnet for its containers
          </li>
        </ul>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://github.com/weaveworks/weave" target="_blank">Weave</a>: overlay network that puts all containers in a distributed system onto a single virtual network; also includes service discovery functionality
          </li>
          <li>
            <a href="http://www.projectcalico.org/" target="_blank">Calico</a>: layer 3 virtual network that provides each container with an IP address
          </li>
        </ul>
      </td>
      
      <td>
      </td>
    </tr>
    
    <tr>
      <td>
        <b>Monitoring / Visibility</b>
      </td>
      
      <td>
        <ul>
          <li>
            Docker <a href="https://docs.docker.com/reference/commandline/ps/" target="_blank">ps</a>/<a href="https://docs.docker.com/reference/commandline/top/" target="_blank">top</a>/<a href="https://docs.docker.com/reference/commandline/stats/" target="_blank">stats</a>: runtime commands
          </li>
          <li>
            <a href="https://docs.docker.com/reference/api/docker_remote_api/" target="_blank">Docker stats API</a>: remote API for streaming basic container metrics; utilized by the Docker <a href="https://blog.docker.com/2015/06/etp-monitoring/" target="_blank">Ecosystem Technology Partners for Monitoring</a>
          </li>
        </ul>
      </td>
      
      <td>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="http://www.sysdig.org/" target="_blank">sysdig</a>: CLI for deep system/containers visibility; includes curses-based “csysdig” interface
          </li>
          <li>
            <a href="https://github.com/google/cadvisor" target="_blank">cAdvisor</a>: basic container metrics exporter from Google; includes web GUI; <a href="https://github.com/kubernetes/heapster" target="_blank">Heapster</a> adds Kubernetes support
          </li>
          <li>
            <a href="https://github.com/weaveworks/scope" target="_blank">Weave Scope</a>: container network topologies
          </li>
        </ul>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://sysdigrp2rs.wpengine.com/" target="_blank">Sysdig Cloud</a>: uses sysdig; includes web-based UI, application topologies, and support for all major container formats and orchestration platforms
          </li>
        </ul>
      </td>
    </tr>
    
    <tr>
      <td>
        <b>Data layer</b>
      </td>
      
      <td>
      </td>
      
      <td>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://clusterhq.com/" target="_blank">CusterHQ Flocker</a>: data volume manager for running stateful services like databases in containers
          </li>
        </ul>
      </td>
      
      <td>
      </td>
    </tr>
    
    <tr>
      <td>
        <b>Log management</b>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://docs.docker.com/reference/commandline/logs/" target="_blank">Docker logs</a>: runtime command
          </li>
        </ul>
      </td>
      
      <td>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://github.com/gliderlabs/logspout" target="_blank">logspout</a>: log router for Docker containers
          </li>
        </ul>
      </td>
      
      <td>
      </td>
    </tr>
    
    <tr>
      <td>
        <b>CI/CD</b>
      </td>
      
      <td>
      </td>
      
      <td>
      </td>
      
      <td>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://app.shippable.com/" target="_blank">Shippable</a>
          </li>
          <li>
            <a href="http://wercker.com/" target="_blank">Wercker</a>
          </li>
        </ul>
      </td>
    </tr>
    
    <tr>
      <td>
        <b>Security</b>
      </td>
      
      <td>
      </td>
      
      <td>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://github.com/OpenSCAP/container-compliance" target="_blank">OpenSCAP</a>
          </li>
        </ul>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://www.twistlock.com/" target="_blank">Twistlock</a>
          </li>
          <li>
            <a href="http://scalock.com/" target="_blank">Scalock</a>
          </li>
          <li>
            <a href="http://www.conjur.net/" target="_blank">Conjur</a>
          </li>
          <li>
            <a href="https://cisofy.com/lynis/plugins/docker-containers" target="_blank">Lynis</a>
          </li>
        </ul>
      </td>
    </tr>
    
    <tr>
      <td>
        <b>Getting started aides</b>
      </td>
      
      <td>
        <ul>
          <li>
            <a href="https://www.docker.com/docker-kitematic" target="_blank">Docker Kitematic</a>: basic Docker GUI designed for getting started with Docker
          </li>
          <li>
            <a href="https://www.docker.com/toolbox" target="_blank">Docker Toolbox</a>: installer for a package of core Docker tools
          </li>
        </ul>
      </td>
      
      <td>
      </td>
      
      <td>
      </td>
      
      <td>
      </td>
    </tr>
  </table>
</div>

## About the Container Ecosystem Project {#about}

Here at <a href="https://sysdigrp2rs.wpengine.com/" target="_blank">Sysdig</a>, the container-native visibility company, we talk to a lot of people in the container ecosystem: both consumers and producers of technology. And wow, there is a LOT of cool technology out there - and so much more coming out all the time. It can be hard to keep up with, even if you’re a seasoned expert, much less as a curious newcomer just trying to figure out where to start. There are plenty of great guides out there for various container technologies and use cases (see below for some links). But we had yet to find a clearly organized survey of the different core technologies that make up the container ecosystem and the typical microservices stack. So we decided to make one: the Container Ecosystem Project. 

The goal of this project is to clearly lay out the different core technologies that might be important for anyone interested in containers and microservices - starting from the lowest levels, and rising up through layers of abstraction to full-blown container platforms. For each type of technology (broken into rows), we’ve tried to provide a brief description (see the left column), as well as list examples currently available for that technology (see the other columns). We’ve separated out open source solutions from commercial offerings, and two of the leading open source container technology producers, Docker and CoreOS, each got their own column. Throughout the doc, we’ve tried to mark beta technologies and parent technologies accordingly. Ideally, this document can introduce you to the microservices stack, and give you some keywords that you can then go research further on your own to learn more - but at least you’ll hopefully have the big picture from here. 

This framework is not, of course, a perfect science, but we’ve done our best to create <a href="https://en.wikipedia.org/wiki/MECE_principle" target="_blank">MECE</a> categories by row, and to put each technology in the most appropriate row. We are almost certainly missing many great technologies, and many technologies listed here do not yet have perfect descriptions. This will be a work in progress. If you have any suggested edits, please <a href="https://twitter.com/sysdig" target="_blank">tweet us</a> or [submit a pull request][1]. We’ll do our best to keep this document up to date and prune off deprecated or abandoned technologies as the ecosystem evolves. 

That’s all for now. I hope this can be a useful resource for the community! 

Update: you can also comment on Hacker News <a href="https://news.ycombinator.com/item?id=10425099" target="_blank">here</a>. 

## Further Reading {#readings}

*   Docker ecosystem introduction from Digital Ocean: <a href="https://www.digitalocean.com/community/tutorial_series/the-docker-ecosystem" target="_blank">https://www.digitalocean.com/community/tutorial_series/the-docker-ecosystem</a>
*   Lists of Docker ecosystem technologies 
    *   <a href="https://www.mindmeister.com/389671722/runc-open-container-ecosystem" target="_blank">https://www.mindmeister.com/389671722/runc-open-container-ecosystem</a>
    *   <a href="https://github.com/weihanwang/docker-ecosystem-survey" target="_blank">https://github.com/weihanwang/docker-ecosystem-survey</a>
    *   <a href="https://github.com/veggiemonk/awesome-docker" target="_blank">https://github.com/veggiemonk/awesome-docker</a>
*   Docker docs: <a href="https://docs.docker.com/" target="_blank">https://docs.docker.com/</a>
*   CoreOS docs: <a href="https://coreos.com/docs/" targte="_blsnk">https://coreos.com/docs/</a>

 [1]: https://github.com/draios/sysdig-container-ecosystem
 [2]: #about
 [3]: #core
 [4]: #distributed
 [5]: #platform
 [6]: #support
 [7]: #readings