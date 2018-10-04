---
ID: 5489
post_title: >
  The Definitive Guide to Container
  Security Terminology
author: Knox Anderson
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/container-security-terminology/
published: true
post_date: 2017-12-12 08:53:40
---
The migration to containers, like every major virtualization shift brings many advantages to your business. Your teams can move quicker, reduce cost by getting higher utilization out of your infrastructure, and can deliver more stable and secure experiences to customers. However, like any other major migration there’s a new set of operational requirements and procedures to follow. We created this guide to Container Security Terminology to make sure you’ve got the *need to know* covered and understand the impact of each layer of security in your environment.

We’ll cover each of the categories below, the key terms, and why each term matters to your security operations.

*   [Operating System Security Terminology][1]

*   [Container Security Terminology][2]

*   [Kubernetes Security Terminology][3]

*   [Container Security Benchmarking and Compliance][4]

## <a name="OperatingSystemSecurityTerminology"></a>Operating System Security Terminology {#anameoperatingsystemsecurityterminologyaoperatingsystemsecurityterminology}

**Kernel Namespaces** (process namespace, user namespace, network namespace) - are a feature of the Linux **kernel** that isolate and virtualize system resources of a collection of processes. Examples of resources that can be virtualized include process IDs, hostnames, user IDs, network access, interprocess communication, and filesystems.

*   **Why you should care:** Namespaces are one of the key functionalities of the Linux kernel that allow containers to exist. It’s the core piece of technology that provides visibility isolation for your containers. You should always be aware and alerted if there is any non authorized modification to existing container namespaces.

**Control Groups (CGroups) -** Control groups or *cgroups* are a feature of the Linux kernel that allow you to limit access processes and containers have to system resources.

*   **Why you should care:** Control groups is the technology that resource isolation has been built upon. They allow resources to be limited for a specific container/process allowing you to safely deploy containers on your infrastructure.

**.scap files -** Similar to a .pcap file a .scap contains a raw output of events that occurred over a point in time. However, a .scap file contains all system events (system calls) rather than just network packets. These can be opened with tools like Sysdig Inspect, sysdig, or even Wireshark for deep post mortem analysis & forensics.

*   **Why you should care:** Rather than using multiple tools like TCPDump, Logs, strace and others you can generate a .scap file with sysdig to get full file, system, network, user, and application data in an easy to consume file that can be analyzed outside of production.

**MAC: Mandatory Access Control -** A type of security where the operating system limits the capabilities of a subject to modify an object. In practice, a subject is usually a process or thread; objects are constructs such as files, directories, <a href="https://en.wikipedia.org/wiki/Transmission_Control_Protocol" target="_blank">TCP</a>/<a href="https://en.wikipedia.org/wiki/User_Datagram_Protocol" target="_blank">UDP</a> ports, shared memory segments, IO devices, etc. Subjects and objects each have a set of security attributes.

*   **Why you should care:** Docker leverages Linux kernel security facilities, such as namespaces, cgroups and Mandatory Access Control, to guarantee an effective isolation of containers.

**SELinux -** a set of kernel capabilities and user based tools that allow administrators to define access policies for the hosts.

*   **Why you should care:** SELinux is one of the ways to add security to your environment with a rich set of default policies. There is documentation and support for general linux security, but a significant amount of modifications will need to be made to policies to support containerized and orchestrated infrastructures. Here’s a post we did comparing <a href="https://sysdigrp2rs.wpengine.com/blog/selinux-seccomp-falco-technical-discussion/" target="_blank">SElinux, Seccomp, and Sysdig Falco</a> and how they do detection and enforcement.

**Seccomp -** is a mechanism in the Linux kernel that allows a process to make a one-way transition to a restricted state where it can only perform a limited set of system calls. If a process attempts any other system calls, it is killed via a SIGKILL signal. In its most restrictive mode, seccomp prevents all system calls other than read(), write(), _exit(), and sigreturn().

*   **Why you should care:** Although restrictive and undoubtedly very secure, seccomp’s strict mode is…strict. You can’t do much other than read/write to already open files. Network activity, starting threads, even getting the current time via gettimeofday(), etc. are all blocked.

**AppArmor -** AppArmor proactively protects threats by enforcing good behavior on your systems through a kernel module and other user space tools.

*   **Why you should care:** AppArmor is another solid opensource option for protecting your hosts and has been extended to include default <a href="https://cloud.google.com/container-optimized-os/docs/how-to/secure-apparmor" target="_blank">container rulesets</a> as well.

## <a name="ContainerSecurityTerminology"></a>Container Security Terminology {#anamecontainersecurityterminologyacontainersecurityterminology}

**Resource limits (CPU, memory) and requests -** Docker can enforce both hard memory limits and soft memory limits. Hard limits allow the container to use no more than an allocated amount of user or system memory. Soft limits allow the container to use as much memory as needed until there are contentions with the underlying host. The flipside of this are resource requests, which guarantee your container will have the necessary resources to operate effectively.

*   **Why container resource limits matter:** User or programs can commit resource abuse if limits aren’t enforced on containers. They’ll be able to consume unlimited amounts of host resources and causing the kernel to throw OOM exceptions and begin killing processes on your system. You can learn more about setting docker resource limits <a href="https://docs.docker.com/engine/admin/resource_constraints/" target="_blank">here</a>.

**Resource Abuse -** Containers are typically deployed at a higher density than VMs. They are lightweight and you can spawn large clusters of them on modest hardware. That’s definitely an advantage, but it implies that a lot of entities are competing for the host resources.

*   **Why you should care:** Software bugs, sizing miscalculations, or a deliberate malware attack can easily cause a *Denial of Service* if you don’t properly configure resource limits.

**Container Breakouts -** When a container has bypassed isolation checks, accessing sensitive information from the host or gaining additional privileges

*   **<a href="https://sysdigrp2rs.wpengine.com/blog/container-isolation-gone-wrong/" target="_blank">What can happen when a container loses isolation</a>:** This means the process running inside the container is able to access anything outside the namespace meaning they can access anything else running inside other containers or on the host. With this information and the kernel capability it is possible to walk the host’s filesystem tree until you find the object you wish to open and then extract sensitive information like passwords.

[tweet_box design="default" float="none"]Container Breakouts & 30+ other container security terms - what they mean & and why they matter[/tweet_box] 

**Image Scanning (static analysis) -** A procedure in which a scanning provider analyzes each layer of the container image, indexes it, and compares the data against a Common Vulnerabilities and Exposures (CVE) database.

*   **Why you should scan your images:** Image scanning provides a way to easily track what you’ve put into your environment and have multiple checks to spot vulnerabilities when your images are in your CI/CD pipeline before they hit production. Check out this post for some <a href="https://sysdigrp2rs.wpengine.com/blog/20-docker-security-tools/" target="_blank">great free scanning providers</a>.

**Certified Images -** Images that have been verified to contains specific contents (data) from a trusted publisher.The <a href="https://store.docker.com/search?certification_status=certified&q=&source=verified&type=image" target="_blank">Docker Store</a> also has a list of certified containers that developers can pull into their environments.

*   **Using certified images:** There are multiple SLAs that certified images have to meet about vulnerability protection and response to vulnerabilities found. These images have been certified by Docker with <a href="https://success.docker.com/store" target="_blank">strict guidelines</a> about updating images based on vulnerabilities.

**Signed Images -** You can use trust to do client-side certification of any image you’re pulling from a registry.

*   **Uses for container image signing:** Image signing gives you the ability to guarantee an image hasn’t been altered since it was signed. It can be used to prevent images from starting that haven’t been signed.

**Privileged Container -** A feature within Docker that allows a container to have access to the host (root).

*   **Why you should care:** Any container that is running as a privileged container has root access to the host. These should be monitored carefully if introduced to your environment, and only allowed for specific images and users.

**Docker Daemon -** the persistent process that manages containers.

*   **Why you should care:** The docker daemon is the process that runs all the containers on the host. There are many capabilities built in to protect it from resource abuse, and other vulnerabilities.

**Docker Daemon Socket -** The way that the Docker daemon can listen to Docker API requests. These can come in the form of unix, tcp, and fd. If any type of remote access is needed to the Docker Daemon you’ll need to enable the tcp socket.

*   **How to safely communicate with the docker daemon:** The Docker Daemon Socket is how internal or 3rd party tools communicate with the Docker Engine API. By default the tcp socket provides unauthenticated and unencrypted access to do the Docker API. This is one of the first things <a href="https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-socket-option" target="_blank">you should modify</a>, and understand if you’re new to a container environment.

**Docker Commit -** the ability to create a new image from the changes to a container.

*   **When to commit a container:** If changes have been made to the filesystem of a container by a user or system this will give you the ability to create an image from those changes, compare them to existing version of that image, or recreate scenarios based on changes that were made.

**Docker Artifacts -** The output from the build process including all dependencies, libraries, needed to build the image, and the result of the build. 

*   **Using Docker Artifacts:** Artifacts give you the complete history of different builds and what was included. Giving an audit trail and the ability to compare current versions with past versions of any image. For non dockerized environments this is typically your JAR file.

**Sysdig Falco -** an open source, container security monitor designed to detect anomalous activity in your applications. Falco lets you continuously monitor and detect container, application, host, and network activity, from one source of data, with one set of customizable rules.

*   **Where to use Sysdig Falco:** Falco is an opensource run-time container security solution that was built specifically for containerized environments and has a default ruleset that’s being managed and improved by the opensource community and companies like yahoo & cloud.gov

**Container Run-time security -** Looking at real-time activity of containers, network, system, etc to detect and block unexpected or unwanted activity or behavior on hosts or inside your containers. This is separate from static scanning or continuous scanning which just looks for vulnerabilities in containers or on hosts.

*   **Why you should care:** The isolation and portability of containers also make it very difficult to see what’s actually going on inside of them. Run-time security provides a layer of visibility and allows you to detect unwanted activities/behaviors from users and programs, and then take actions like stopping or pausing a container to block the attack.

**Zero-day vulnerability -** An exposure in a piece of software that is unknown to the vendor or maintainer. These can later be exposed by 3rd parties to take down a service, steal data, or other malicious actions.

*   **Why containers may help limit the impact of zero day vulnerabilities:** Zero day vulnerabilities are often thought to be impossible to protect against because they are unknown to the maintainer of the software. The process isolation of a container combined with newer security approaches that baseline at container activity could lead to zero day threats being detected as as soon as they’re exploited.

## <a name="KubernetesSecurityTerminology"></a><a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-harden-kube-system/" target="_blank">Kubernetes Security Terminology</a> {#anamekubernetessecurityterminologyakubernetessecurityterminologyhttpssysdigcomblogkubernetessecurityhardenkubesystem}

**RBAC -** Role-Based Access Control ("RBAC") uses the “rbac.authorization.k8s.io” API group to drive authorization decisions, allowing admins to dynamically configure policies through the Kubernetes API.

*   **Why you should care:** This allows you to further configure access policies at a user or group level to prevent internal actors from getting access to services that they shouldn’t.

**Kubernetes Transport Level Security (TLS) -** this protocol aims primarily to provide privacy and data integrity between two communicating applications

*   **Why you should care:** Just like standard TLS connections the kubernetes tls protocol aims to protect you by validating the identity of a server, encrypting connections, and verifying the integrity and the authenticity of connection

**<a href="https://kubernetes.io/docs/admin/authorization/node/" target="_blank">Node Authorizer</a> -** The Node authorizer allows a kubelet to perform API operations. This includes:

*   **Read Operations -** used for pulling metadata or metrics

*   **Write operations -** can add a NodeRestriction to limit what the kubelet can modify

*   **Auth operation -** used for auth & certification checks

*   **Why you should care:** There are multiple settings and plugins for the Node Authorizer to help lock down the security of Kubernetes internals.

**Pod security policy -** A pod security policy lists the conditions a pod must meet in order to run in the cluster

*   **Why you should care:** Pod security policies are one of the first lines of security you should add to your kubernetes cluster. They can do simple things like stopping the launch of privileged containers to disallow containers that access volumes apart from NFS volumes.

**Taints -** Taints are a way to label a node as no-schedule.

*   **Why you should care:** Master nodes in particular are areas of your kube cluster where you would want to make sure no other resources get scheduled on that node.

**Kubernetes Network Policy Provider -** The ability to specify how a collection of pods are allowed to talk to each other or other API endpoints.

*   **Why you should care:** The network policy provider is a way to manage ingress/egress security policies internally through kubernetes.

## <a name="ContainerSecurityBenchmarkingandCompliance"></a>Container Security Benchmarking and Compliance {#anamecontainersecuritybenchmarkingandcomplianceacontainersecuritybenchmarkingandcompliance}

**CIS Docker Benchmark -** The CIS (Center for Internet Security) is a community driven non-profit that drives best practices to protect public and private companies from threats. The CIS community has been working with docker ecosystem members to establish guidelines for securing containerized environments.

*   **Why you should care:** The CIS provides extensive reports, guidelines, and checklists to look at while architecture container environments so your infrastructure is Secure by Design.

**Grafeas** - An open artifact metadata API to audit and govern your software supply chain.

*   **Why you should care:** Grafeas is a relatively new project backed by google and a host of other companies designed to make auditing and governance much easier through managing your artificats and metadata. This is a project to keep your eye on as it keeps evolving.

**GDPR -** The <a href="https://sysdigrp2rs.wpengine.com/blog/devops-gdpr-compliance/" target="_blank">European General Data Protection Regulation</a> (GDPR) is a regulation that’s meant to unify and strengthen data protection for all members of the EU. It’s focused around the PII data of European citizens and making sure it is stored properly, and if a breach occurs that certain procedures are met in a timely manner.

*   **<a href="https://sysdigrp2rs.wpengine.com/blog/devops-gdpr-compliance/" target="_blank">Why you should care</a>:** If your company is doing any business in Europe where you are collecting any type of PII data then you will need to be compliant with GDPR. Penalties can be up to 4% of your company's overall revenue or 20 million euros.

**Common vulnerability & exposure (CVE) database -** A publicly managed database of known information security vulnerabilities and exposures that software providers can use to compare with what is currently running in their environment.

*   **Why you should care:** This is a checkbox categories in your security posture where you know that the software you are using doesn’t have known vulnerabilities before you push it to production.

**NIST -** NIST develops and issues guidelines to assist federal agencies in implementing (FISMA) and to help with managing cost effective programs to protect their information and information systems.

*   **Why you should care:** if you’re a government agency this is one of the best resources to help improve your security best practices.

**FedRamp -** The Federal Risk and Authorization Management Program, or FedRAMP, is a government-wide program that provides a standardized approach to security assessment, authorization, and continuous monitoring for cloud products and services.

*   **Why you should care:** There mantra of "do once, use many times" is a good one to follow for security procedures in your environment. Looking at their program especially if you’re a government agency can provide huge benefits when going through security tool selection and validation processes.

### Help out the community {#helpoutthecommunity}

Like any list, this guide isn’t exhaustive and is our first pass at adding what we think the new and existing container operators need to know from a security perspective! If there is something you think we missed or a category that you’d like us to add please tweet us @sysdig

See how <a href="https://sysdigrp2rs.wpengine.com/product/secure/" target="_blank">Sysdig Secure</a> can provide run-time security and forensics for you containers. If you still need to do some more research check out our whitepaper comparing <a href="https://go.sysdigrp2rs.wpengine.com/20-docker-tools-compared" target="_blank">20+ opensource and commercial docker security tools</a>.

 [1]: #OperatingSystemSecurityTerminology
 [2]: #ContainerSecurityTerminology
 [3]: #KubernetesSecurityTerminology
 [4]: #ContainerSecurityBenchmarkingandCompliance