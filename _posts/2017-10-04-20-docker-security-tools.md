---
ID: 4402
post_title: 20 Docker security tools compared
author: Mateo Burillo
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/20-docker-security-tools/
published: true
post_date: 2017-10-04 18:30:52
---
### **UPDATE!**

**We strive to provide the most complete and up-to-date list of Docker security tools. We are keeping the number "20" in the title, but the list has 22 items at this moment... and growing.**

There are quite a few Docker security tools in the ecosystem, how do they compare? This is a comprehensive list of Docker security tools that can help you implement some of the container security best practices.

**Is Docker insecure?** Not at all. Actually features like process isolation with user namespaces, resource encapsulation with cgroups, immutable images and shipping the minimal software and dependencies reduce the attack vector providing a great deal of protection. But, is there anything else we can do? There is much more than image vulnerability scanning and these are 20 container and Docker specific security tools that can help. [adrotate banner="25"] 

## Alphabetical index of Docker Security tools

*   [Anchore Navigator][1]
*   [AppArmor][2]
*   [AquaSec][3]
*   [BlackDuck Docker security][4]
*   [Cavirin][5]
*   [Cilium][6]
*   [CoreOS Clair][7]
*   [Docker capabilities and resource quotas][8]
*   [Docker-bench security][9]
*   [Dockscan][10]
*   [**Falco**][11]
*   [HashiCorp Vault][12]
*   [NeuVector][13]
*   [Notary][14]
*   [OpenSCAP][15]
*   [REMnux][16]
*   [SELinux][17]
*   [Seccomp][18]
*   [StackRox][19]
*   [**Sysdig Secure**][20]
*   [Sysdig][21]
*   [Tenable Flawcheck][22]
*   [Twistlock][23] [tweet_box design="default" float="none"]22 #Docker #security tools list (and growing). It’s dangerous out there, be prepared![/tweet_box] 

## <a id="Anchore-Navigator"></a>Anchore Navigator

  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/anchore.png" alt="anchore navigator" style="float: right; margin: 0 1em 1em 0;" /> 
**Homepage:** <a href="https://anchore.io/" target="_blank" rel="noopener">https://anchore.io/</a>

**License:** Commercial, some services are free to use.

**Use Cases:** Pre-production analysis, vulnerability newsfeed.

<p style="clear: right; display: block;">
  Anchore Navigator provides a free service for deep inspection of public Docker images. You can also explore their rich repository of already-dissected images for full visibility of its content, build process, and discovered CVE threats together with a link to the complete issue description and known fixes.
</p>

Using this tool you can perform a thorough analysis of your own images and subscribe to the images you frequently use for your deployments to receive timely security warnings.

  
## <a id="AppArmor"></a>AppArmor

  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/apparmor.png" alt="apparmor" style="float: right; margin: 0 1em 1em 0;" /> 
**Homepage:** <a href="http://wiki.apparmor.net" target="_blank" rel="noopener">http://wiki.apparmor.net</a>

**License:** Open Source.

**Use Cases:** Runtime protection, Mandatory Access Control (MAC).

<p style="clear: right; display: block;">
  AppArmor lets the administrator assign a security profile to each program in your system: filesystem access, network capabilities, link and execute rules, etc.
</p>

It’s a Mandatory Access Control (or MAC) system, meaning that it will prevent the forbidden action from taking place, although it can also report profile violation attempts.

AppArmor it’s sometimes considered a more accessible and simplified version of [SELinux][17], both are closely related. You only need to learn the <a href="http://wiki.apparmor.net/index.php/QuickProfileLanguage" target="_blank" rel="noopener">profile language syntax</a> and fire your favorite editor to start writing your own AppArmor rules.

**Docker context:** Docker can automatically generate and load a default AppArmor profile for containers named docker-default. You can create <a href="https://github.com/docker/labs/blob/master/security/apparmor/README.md" target="_blank" rel="noopener">specific security profiles</a> for your containers or the applications inside them.

  
## <a id="AquaSec"></a>AquaSec

  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/aquasec.png" alt="aquasec" style="float: right; margin: 0 1em 1em 0;" /> 
**Homepage:** <a href="https://www.aquasec.com/" target="_blank" rel="noopener">https://www.aquasec.com/</a>

**License:** Commercial.

**Use Cases:** Pre-production analysis, runtime protection, compliance & audit, etc.

<div class="page" title="Page 6">
  <div class="section">
    <div class="layoutArea">
      <div class="column">
        <p>
          <span>AquaSec is a commercial security suite designed for containers in mind. Security audit, container image verification, runtime protection, automated policy learning or intrusion prevention capabilities are some of the most relevant features. </span>
        </p>
        
        <p>
          <span>The platform provides programmatic access to its API and can be deployed both locally or in the public cloud. </span>
        </p>
      </div>
    </div>
  </div>
</div>

  
## <a id="BlackDuck-Docker-security"></a>BlackDuck Docker Security

  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/blackduck.png" alt="blackduck" style="float: right; margin: 0 1em 1em 0;" /> 
**Homepage:** <a href="https://www.blackducksoftware.com/solutions/container-security" target="_blank" rel="noopener">https://www.blackducksoftware.com</a>

**License:** Commercial.

**Use Cases:** Pre-production analysis, vulnerability newsfeed, license/legal assessment.

<p style="clear: right; display: block;">
  Black Duck Hub specializes in container inventory and reporting image inventory, mapping known security vulnerabilities to images indexes and cross project risk reports. You can easily pinpoint the specific libraries, software packages or binaries that are causing the security risk and the assistant will automatically offer you a list of known fixes.
</p>

As opposed to similar solutions, Black Duck Hub also analyzes the “License Risk” considering the different software licences that you are currently bundling together to deploy your containerized distributed system.

<div class="page" title="Page 7">
  <div class="section">
    <div class="layoutArea">
      <div class="column">
        <p>
          <span>BlackDuck focuses more on scanning and pre-production than run- time security and forensics. </span>
        </p>
      </div>
    </div>
  </div>
</div>

  
## <a id="Cilium"></a>Cilium

  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/cilium.png" alt="cilium" style="float: right; margin: 0 1em 1em 0;" /> 
**Homepage:** <a href="https://www.cilium.io/" target="_blank" rel="noopener">https://www.cilium.io/</a>

**License:** Open Source.

**Use Cases:** HTTP-layer security, network-layer security.

<p style="clear: right; display: block;">
  Cilium provides transparent network security between container applications. Based on a new Linux kernel technology called eBPF, it allows you to define and enforce both network-layer and HTTP-layer security policies based on container/pod identity.
</p>

Cilium leverages BPF to perform core data path filtering, mangling, monitoring and redirection. These BPF capabilities are available in any Linux kernel version 4.8.0 or newer.

  
## <a id="Cavirin"></a>Cavirin

  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/cavirin.png" alt="cavirin" style="float: right; margin: 0 1em 1em 0;" /> 
**Homepage:** <a href="https://cavirin.com" target="_blank" rel="noopener">https://cavirin.com</a>

**License:** Commercial.

**Use Cases:** Runtime protection, pre-production analysis, compliance & audit

<p style="clear: right; display: block;">
  Cavirin works with organizations such as <a href="https://www.cisecurity.org/" target="_blank" rel="noopener">CIS</a> to collaboratively develop and maintain the security standards that any other tool can benefit from. At present, it has authored CIS Docker Security Benchmark as well as CIS Kubernetes Security Benchmark. They have minted the term "DevSecOps" to stress their focus at integrating the security and DevOps/container fields. Apart from the features you can expect in a one-stop DevOps security platform (maybe comparable to Twistlock or AquaSec in their feature proposal and approach), we can highlight their compliance&audit tooling for security standards like PCI, HIPAA, NIST or GDPR.
</p>

  
## <a id="CoreOS-Clair"></a>CoreOS Clair

  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/clair.png" alt="clair" style="float: right; margin: 0 1em 1em 0;" /> 
**Homepage:** <a href="https://coreos.com/clair/docs/latest/" target="_blank" rel="noopener">https://coreos.com/clair/docs/latest/</a>

**License:** Open Source.

**Use Cases:** Pre-production analysis, vulnerability newsfeed.

<p style="clear: right; display: block;">
  Clair is an open source project for the static analysis of vulnerabilities in containers (currently supporting AppC and Docker). Clair periodically refreshes its vulnerability database from a set of configured CVE sources, scrubs the available container images and indexes the installed software packages. If any insecure software is detected, it can alert or block deployment to production.
</p>

Since Clair image analysis is static, containers never need to be actually executed, so you can detect a security threat before is already running in your systems. Clair is the security engine that CoreOS Quay registry uses internally.

  
## <a id="Docker-capabilities-and-resource-quotas"></a>Docker capabilities and resource quotas

  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/docker.png" alt="docker" style="float: right; margin: 0 1em 1em 0;" /> 
**Homepage:** <a href="https://www.docker.com/community-edition" target="_blank" rel="noopener">https://www.docker.com</a>

**License:** Open Source.

**Use Cases:** Runtime protection, resource DoS protection.

<p style="clear: right; display: block;">
  We shouldn’t forget the basic security measures that come already bundled with our OS and the Docker engine.
</p>

Resource abuse and denial of service is an often overlooked but very real security problem in a containerized environment with vast amounts of software entities competing for the host resources.

Control Groups (cgroups) is a feature of the Linux kernel that allows you to <a href="https://github.com/docker/labs/blob/master/security/cgroups/README.md" target="_blank" rel="noopener">limit the access</a> processes and containers have to system resources such as CPU, RAM, IOPS and network.

Capabilities allows you to break down the full root permissions into several split permissions, this way you can <a href="https://github.com/docker/labs/blob/master/security/capabilities/README.md" target="_blank" rel="noopener">remove specific capabilities</a> from the root account or augment the capabilities of user accounts at a more granular level.

  
## <a id="Docker-bench-security"></a>Docker-bench security

  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/docker-bench.png" alt="docker-bench" style="float: right; margin: 0 1em 1em 0;" /> 
**Homepage:** <a href="https://github.com/docker/docker-bench-security" target="_blank" rel="noopener">https://github.com/docker/docker-bench-security</a>

**License:** Open Source.

**Use Cases:** Compliance & security audit.

<p style="clear: right; display: block;">
  The Docker Bench for Security is a meta-script that checks for dozens of common best-practices around deploying Docker containers in production.
</p>

This script is conveniently packaged as a Docker container, just copying and pasting the docker run one-liner from its homepage you can instantly see the results of ~250 checks for your running Docker containers and the host running the Docker engine (Docker CE or Docker Swarm). Docker Bench tests are inspired by the <a href="https://www.cisecurity.org/cis-benchmarks/" target="_blank" rel="noopener">CIS Docker Community Edition Benchmark v1.1.0.</a>

  
## <a id="Dockscan"></a>Dockscan

  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/dockscan.png" alt="dockscan" style="float: right; margin: 0 1em 1em 0;" /> 
**Homepage:** <a href="https://github.com/kost/dockscan" target="_blank" rel="noopener">https://github.com/kost/dockscan</a>

**License:** Open Source.

**Use Cases:** Compliance & audit.

<p style="clear: right; display: block;">
  A simple ruby script that analyzes the Docker installation and running containers, both for local and remote hosts.
</p>

It’s easy to install and run with just one command and can generate HTML report files. Dockscan reports configured resource limits, containers spawning too many processes or with a high number of modified files, also if your Docker host is allowing containers to directly forward traffic to the host gateway, to name a few examples.

  
## <a id="Falco"></a>Falco

  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/falco.png" alt="falco" style="float: right; margin: 0 1em 1em 0;" /> 
**Homepage:** <a href="https://www.sysdig.org/falco/" target="_blank" rel="noopener">https://www.sysdig.org/falco/</a>

**License:** Open Source.

**Use Cases:** Runtime alerting, forensics.

Example of blog post edit.

<p style="clear: right; display: block;">
  Sysdig Falco is an open source, behavioral monitoring software designed to detect anomalous activity based on the <a href="#Sysdig">Sysdig</a> monitoring technology. Sysdig Falco also works as a intrusion detection system on any Linux host.
</p>

Falco is an auditing tool as opposed to enforcement tools like <a href="https://sysdigrp2rs.wpengine.com/blog/selinux-seccomp-falco-technical-discussion/" target="_blank" rel="noopener">Seccomp or AppArmor</a>. It runs in user space, using a kernel module to retrieve system calls, while other similar tools perform system call filtering/monitoring at the kernel level. One of the benefits of a user space implementation is being able to integrate with external systems like Docker, Docker Swarm, Kubernetes, Mesos, etc and incorporate their metadata and tags.

**Docker context:** Falco supports container-specific context for its rules. Using this tool you can monitor the containers behaviour without instrumenting or modifying them in any way. Custom rule creation is very easy to grasp and the default rules file comes prepopulated with sane defaults.

  
## <a id="HashiCorp-Vault"></a>HashiCorp Vault

  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/vault.jpg" alt="vault" style="float: right; margin: 0 1em 1em 0;" /> 
**Homepage:** <a href="https://www.vaultproject.io/" target="_blank" rel="noopener">https://www.vaultproject.io/</a>

**License:** Free with enterprise version.

**Use Cases:** Secure container-aware credentials storage, trust management.

<p style="clear: right; display: block;">
  Hashicorp’s Vault is an advanced suite for managing secrets: Passwords, SSL/TLS certificates, API keys, access tokens, SSH credentials, etc. It supports time-based secret leases, fine-grained secret access, on-the-fly generation of new secrets, key rolling (renewing keys without losing access to secrets generated using the old one) and much more.
</p>

Vaults keeps a detailed audit log to keep track of all the secrets and the access and manipulations performed by each user/entity, so operators can easily trace any suspicious interaction.

**Docker context:** The secure distribution and traceability of secrets is a core concern in the new microservices and containerized environments, where software entities are constantly spawned and deleted. Vault itself can be deployed as a Docker container.

  
## <a id="NeuVector"></a>NeuVector

  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/neuvector.png" alt="neuvector" style="float: right; margin: 0 1em 1em 0;" /> 
**Homepage:** <a href="http://neuvector.com/" target="_blank" rel="noopener">http://neuvector.com/</a>

**License:** Commercial.

**Use Cases:** Runtime protection, compliance & audit.

<div class="page" title="Page 16">
  <div class="section">
    <div class="layoutArea">
      <div class="column">
        <p>
          <span>NeuVector focuses on real-time security protection at runtime. Automatically discovers behavior of applications, containers, and services, detects security escalations and other related threats in a similar fashion to other Linux IDS. NeuVector privileged ‘enforcer’ containers are deployed on each physical host, with full access to the local Docker daemon, apart from that, the internal technology used by NeuVector is not thoroughly detailed in the publicly accessible documentation. </span>
        </p>
        
        <p>
          <span>NeuVector aims to be a non-intrusive, plug&play security suite, performing automatic discovery of running containers and their default behavior to assist and counsel the operators in the design of their infrastructure security profiles. NueVector focuses on container network security rather than the underlying system like many of the other run-time players. </span>
        </p>
      </div>
    </div>
  </div>
</div>

  
## <a id="Notary"></a>Notary

  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/notary.png" alt="notary" style="float: right; margin: 0 1em 1em 0;" /> 
**Homepage:** <a href="https://github.com/docker/notary" target="_blank" rel="noopener">https://github.com/docker/notary</a>

**License:** Open Source

**Use Cases:** Trusted image repository, trust management and verifiability.

<p style="clear: right; display: block;">
  Image forgery and tampering is one major security concern for Docker-based deployments. Notary is a tool for publishing and managing trusted collections of content. You can approve trusted published and create signed collections, in a similar fashion to the software repository management tools present in modern Linux systems, but for Docker images.
</p>

Some of Notary goals include guaranteeing image freshness (most up to date content, to avoid known vulnerabilities), trust delegation between users or trusted distribution over untrusted mirrors or transport channels.

  
## <a id="OpenSCAP"></a>OpenSCAP

  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/openscap.png" alt="openscap" style="float: right; margin: 0 1em 1em 0;" /> 
**Homepage:** <a href="https://www.open-scap.org/" target="_blank" rel="noopener">https://www.open-scap.org/</a>

**License:** Open Source.

**Use Cases:** Compliance & audit, certification

<p style="clear: right; display: block;">
  OpenSCAP provides a suite of automated audit tools to examine the configuration and known vulnerabilities in your software, following the NIST-certified Security Content Automation Protocol (SCAP).
</p>

You can create your own custom assertions and rules and routinely check that any software deployed in your organization strictly abides.

These set of tools is not only focused on the security itself, but also on providing the formal tests and reports that you may need to meet an official security standard.

**Docker context:** The OpenSCAP suite provides a Docker-specific tool <a href="https://www.open-scap.org/resources/documentation/security-compliance-of-rhel7-docker-containers/" target="_blank" rel="noopener">oscap-docker</a> to audit your images, assessing both running containers and cold images.

  
## <a id="REMnux"></a>REMnux

  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/remnux.png" alt="remnux" style="float: right; margin: 0 1em 1em 0;" /> 
**Homepage:** <a href="https://remnux.org/" target="_blank" rel="noopener">https://remnux.org/</a>

**License:** Open Source.

**Use Cases:** Forensics.

<p style="clear: right; display: block;">
  A security oriented distribution based on Ubuntu. REMnux is a free Linux toolkit for assisting malware analysts with reverse-engineering malicious software, commonly known as forensics. As you can guess, this system bundles a vast amount of pre installed <a href="https://remnux.org/docs/distro/tools/" target="_blank" rel="noopener">analysis and security tools</a>: Wireshark, ClamAV, tcpextract, Rhino debugger, Sysdig, vivisect… just to name a few.
</p>

REMnux aims to be the swiss knife that you carry around in a usb memory in case you suspect any of your systems have been compromised.

**Docker context:** The REMnux project conveniently provides several of its integrated security tools as <a href="https://remnux.org/docs/containers/malware-analysis/" target="_blank" rel="noopener">Docker containers</a>, so you can instantly launch difficult-to-install security applications when you most need them.

  
## <a id="SELinux"></a>SELinux

  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/selinux.png" alt="selinux" style="float: right; margin: 0 1em 1em 0;" /> 
**Homepage:** <a href="https://selinuxproject.org/page/Main_Page" target="_blank" rel="noopener">https://selinuxproject.org</a>

**License:** Open Source.

**Use Cases:** Runtime protection, Mandatory Access Control (MAC).

<p style="clear: right; display: block;">
  Security-Enhanced Linux (SELinux) is a Linux kernel security module. It is often compared with <a href="#AppArmor">AppArmor</a>, and it’s also a Mandatory Access Control system. SELinux provides security capabilities from mandatory access controls to mandatory integrity controls, role-based access control (RBAC) and type enforcement architecture.
</p>

SELinux has a reputation of being particularly complex but powerful, fine-grained and flexible.

**Docker context:** Similarly to AppArmor, SELinux offers an extra layer of access policies and <a href="https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux_atomic_host/7/html/container_security_guide/docker_selinux_security_policy" target="_blank" rel="noopener">isolation between the host and the containerized apps.</a>

  
## <a id="Seccomp"></a>Seccomp

  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/seccomp.png" alt="seccomp" style="float: right; margin: 0 1em 1em 0;" /> 
**Homepage:** <a href="https://www.kernel.org/doc/Documentation/prctl/seccomp_filter.txt" target="_blank" rel="noopener">https://www.kernel.org</a>

**License:** Open Source.

**Use Cases:** Runtime protection, Mandatory Access Control (MAC).

<p style="clear: right; display: block;">
  Seccomp is not so much a tool but rather a sandboxing facility in the Linux kernel. You can think of it as an iptables rules-based firewall but for system calls. Newer versions use Berkeley Packet Filter (BPF) rules to filter syscalls and control how they are handled.
</p>

With Seccomp you can selectively choose which syscalls are forbidden/allowed to each container. For <a href="https://github.com/docker/labs/tree/master/security/seccomp" target="_blank" rel="noopener">example</a>, you can forbid file-permissions manipulations inside your container.

You may have noticed the similarities with [Falco][11], both are closely related to the Linux Syscall API. This <a href="https://sysdigrp2rs.wpengine.com/blog/selinux-seccomp-falco-technical-discussion/" target="_blank" rel="noopener">article</a> compares these two (with AppArmor and SELinux) solutions. TL;DR: Unlike the others, Falco integrates rich high level container specific context to build rules.

**Docker context:** Docker has used Seccomp since version 1.10 of the Docker Engine, Docker has its own <a href="https://docs.docker.com/engine/security/seccomp/" target="_blank" rel="noopener">JSON-based</a> DSL that allows you to define profiles that will be compiled to seccomp filters.

  
## <a id="Stackrox"></a>StackRox

  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/stackrox.png" alt="stackrox" style="float: right; margin: 0 1em 1em 0;" /> 
**Homepage:** <a href="https://www.stackrox.com" target="_blank" rel="noopener">https://www.stackrox.com</a>

**License:** Commercial.

**Use Cases:** Runtime protection, machine learning, pre-production analysis.

<p style="clear: right; display: block;">
  StackRox feature proposal revolves around the concepts of "Adaptive security" and auto discovery of components and behaviors. Highly focused on machine learning, StackRox aims to provides security that will evolve with your platform.<br />
</p>

StackRox provides the usual features of commercial security platforms like cold image scanning or default security profiles ala SELinux.  


StackRox understands containers and the images in your environment but can't enforce policies based services determined by your orchestrator. They focus more on pre-production and run-time workloads rather than forensics and incident response. 

  
## <a id="Sysdig-Secure"></a>Sysdig Secure

  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/10/Explore-Policy-Violation.png" alt="Sysdig Secure" style="float: right; margin: 0 1em 1em 0;" /> 
**Homepage:** <a href="https://www.sysdigrp2rs.wpengine.com/product/secure" target="_blank" rel="noopener">https://www.sysdigrp2rs.wpengine.com/product/secure</a>

**License: **Commercial

**Use Cases:** Pre-production analysis, vulnerability newsfeed, runtime security, compliance & audit and forensics, hybrid environments (containers and traditional deployment), performance monitoring & troubleshooting, available both as SaaS and on-prem.

<p style="clear: right; display: block;">
</p>

Sysdig Secure is a powerful run-time security and forensics solution for your containers and microservices. Secure is part of the Sysdig Container Intelligence Platform, and as the rest of the family comes out-of-the-box with deep container visibility and container orchestrator tools integration, including Kubernetes, Docker, AWS ECS, and Mesos.  


Sysdig Secure protects your entire infrastructure: containers & hosts as well as the logical services that run on top of them. Sysdig Secure also provides full stack forensics capabilities for pre and post attack investigation.  


Sysdig provides full performance monitoring and troubleshooting for your environment. A single instrumentation both for monitoring and security with no added overhead. 


## <a id="Sysdig"></a>Sysdig

  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/sysdig.png" alt="sysdig" style="float: right; margin: 0 1em 1em 0;" /> 

**Homepage:** <a href="https://www.sysdig.org/" target="_blank" rel="noopener">https://www.sysdig.org/</a>

**License:** Open source, commercial products built on top of the free technology.

**Use Cases:** Anomalous behaviour debugging, forensics.

<p style="clear: right; display: block;">
  Sysdig is a full-system exploration, troubleshooting and debugging tool for Linux systems. It records all system calls made by any process, allowing system administrators to debug the operating system or any processes running on it.
</p>

Sysdig has a command line interface with a syntax similar to tcpdump and a ncurses interface to visually navigate and filter through the events, in a similar fashion to htop or wireshark. The system call capture files allows you to perform forensics on your containers <a href="https://sysdigrp2rs.wpengine.com/blog/fishing-for-hackers/" target="_blank" rel="noopener">even if they are long gone.</a>

  
## <a id="Tenable-Flawcheck"></a>Tenable Flawcheck

  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/flawcheck.png" alt="flawcheck" style="float: right; margin: 0 1em 1em 0;" /> 
**Homepage:** <a href="https://www.tenable.com/flawcheck" target="_blank" rel="noopener">https://www.tenable.com/flawcheck</a>

**License:** Commercial.

**Use Cases:** Pre-production analysis, vulnerability newsfeed.

<p style="clear: right; display: block;">
  Tenable, the company perhaps best know for Nessus, the security scanner, acquired Flawcheck, a specific container-focused security solution.
</p>

FlawCheck, like other commercial tools in this list, stores container images and scans them as they’re built, before they can reach production. FlawCheck leverages Tenable/Nessus know-how and database of vulnerabilities, malware and intrusion vectors and adapts it to containerized and agile CI/CD environments.

  
## <a id="Twistlock"></a>Twistlock

  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/twistlock.png" alt="twistlock" style="float: right; margin: 0 1em 1em 0;" /> 
**Homepage:** <a href="https://www.twistlock.com/" target="_blank" rel="noopener">https://www.twistlock.com/</a>

**License:** Commercial.

**Use Cases:** Pre-production analysis, runtime protection, compliance & audit, etc.

<p style="clear: right; display: block;">
  A commercial security suite built to support containerized environments: vulnerability management, access control, and image scanning based standards compliance.
</p>

Twistlock integrates with your continuous integration / continuous delivery pipeline, providing native plugins for popular tools like Jenkins or TeamCity and callable webhooks, so you can trigger the indexing and scanning process for every build and testing environment. Twistlock is known for their popular scanning technology but their run-time security only enforces actions against containers not their underlying hosts, or orchestrated services. 

  
  
  
We hope you find this Docker security tools list useful. If you have suggestions or additional tools we should add, feel free to ping us at <a href="https://twitter.com/sysdig" target="_blank" rel="noopener">@sysdig</a> or reach us on the <a href="https://slack.sysdigrp2rs.wpengine.com/" target="_blank" rel="noopener">Sysdig community Slack group</a>.

 [1]: #Anchore-Navigator
 [2]: #AppArmor
 [3]: #AquaSec
 [4]: #BlackDuck-Docker-security
 [5]: #Cavirin
 [6]: #Cilium
 [7]: #CoreOS-Clair
 [8]: #Docker-capabilities-and-resource-quotas
 [9]: #Docker-bench-security
 [10]: #Dockscan
 [11]: #Falco
 [12]: #HashiCorp-Vault
 [13]: #NeuVector
 [14]: #Notary
 [15]: #OpenSCAP
 [16]: #REMnux
 [17]: #SELinux
 [18]: #Seccomp
 [19]: #Stackrox
 [20]: #Sysdig-Secure
 [21]: #Sysdig
 [22]: #Tenable-Flawcheck
 [23]: #Twistlock
