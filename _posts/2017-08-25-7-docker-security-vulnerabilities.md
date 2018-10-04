---
ID: 4505
post_title: >
  7 Docker security vulnerabilities and
  threats
author: Mateo Burillo
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/7-docker-security-vulnerabilities/
published: true
post_date: 2017-08-25 09:08:40
---
Docker security: *security monitoring* and *security tools* are becoming hot topics in the modern IT world as the early adoption fever is transforming into a mature ecosystem. Docker security is an unavoidable subject to address when we plan to change how we architect our infrastructure.

Docker comes bundled with some neat security safeguards by default:

  
  
  
  
*   **Docker containers are minimal:** One or just a few running processes, only the strictly required software. Less software means smaller probability of being affected by a vulnerability.

*   **Docker containers are task-specific:** There is a pre-definition of what exactly should be running in your containers, path of the data directories, required open ports, daemon configurations, mount points, etc. Any security-related anomaly is easier to detect than in other multi-purpose systems. This design principle goes hand in hand with the microservices approach, greatly reducing the attack surface.

*   **Docker containers are isolated:** Both from the hosting system and from other containers, thanks to the resource isolation features of the Linux kernel such as cgroups and namespaces. There is a notable pitfall here, the kernel itself is shared between the host and the containers, we will address that later on.

*   **Docker containers are reproducible:** Due to their declarative build systems any admin can easily inspect how the container is built and fully understand every step. It’s very unlikely that you end up with a legacy, patched-up system nobody really wants to configure from scratch again, does this ring a bell? ;) [tweet_box design="default" float="none"]7 #Docker security threats: examples, best-practices and tools to battle them[/tweet_box] 

There are, however, some specific parts of Docker based architectures which are more prone to attacks. In this article we are going to cover **7** fundamental Docker security vulnerabilities and threats.

Each section will be divided into:

*   Threat description: Attack vector and why it affects containers in particular.
*   Docker security best practices: What can you do to prevent this kind of security threats.
*   Proof of Concept Example(s): A simple but easily reproducible exercise to get some firsthand practice.

## Docker vulnerabilities and threats to battle {#docker-vulnerabilities-and-threats-to-battle}

[
Docker host and kernel security][1] 

[
Docker container breakout][2] 

[
Container image authenticity][3] 

[
Container resource abuse][4] 

[
Docker security vulnerabilities present in the static image][5] 

[
Docker credentials and secrets][6] 

[
Docker runtime security monitoring][7] 

## Docker host and kernel security {#docker-host-and-kernel-security}

**Description:** If any attacker compromises your host system, the container isolation and security safeguards won’t make much of a difference. Besides, containers run on top of the host kernel by design. This is really efficient for multiple reasons you probably know already, but from the point of view of security it can be seen as a risk that needs to be mitigated.

**Best Practices:** How to “secure a Linux host” is a huge theme with plenty of literature available. If we just focus on the Docker context:

*   Make sure your host & Docker engine configuration is secure (restricted and authenticated access, encrypted communication, etc). We recommend using the <a href="https://github.com/docker/docker-bench-security" target="_blank">Docker bench audit tool</a> to check configuration best practices.

*   Keep your base system reasonably updated, subscribe to security newsfeeds for the operating system and any software you install on top, especially if it’s coming from 3rd party repositories, like the container orchestration you installed.

*   Using minimal, container-centric host systems like CoreOS, Red Hat Atomic, RancherOS, etc, will reduce your attack surface and can bring some helpful new features like running system services in containers as well.

*   You can enforce *Mandatory Access Control* to prevent undesired operations -both on the host and on the containers- at the kernel level using tools like Seccomp, AppArmor or SELinux.

**Examples:**

Seccomp allows you to filter which actions can be performed by the container, particularly system calls. Think of it as a firewall, but for the kernel call interface.

Some capabilities are already blocked by the default Docker profile, try this:

    <div class="line"><span class="syntax--text syntax--plain"><span class="syntax--meta syntax--paragraph syntax--text"><span>#&nbsp;docker&nbsp;run&nbsp;-it&nbsp;alpine&nbsp;sh</span></span></span></div><div class="line"><span class="syntax--text syntax--plain"><span class="syntax--meta syntax--paragraph syntax--text"><span>/&nbsp;#&nbsp;whoami</span></span></span></div><div class="line"><span class="syntax--text syntax--plain"><span class="syntax--meta syntax--paragraph syntax--text"><span>root</span></span></span></div><div class="line"><span class="syntax--text syntax--plain"><span class="syntax--meta syntax--paragraph syntax--text"><span>/&nbsp;#&nbsp;mount&nbsp;/dev/sda1&nbsp;/tmp</span></span></span></div><div class="line"><span class="syntax--text syntax--plain"><span class="syntax--meta syntax--paragraph syntax--text"><span>mount:&nbsp;permission&nbsp;denied&nbsp;(are&nbsp;you&nbsp;root?)</span></span></span></div>

Or

<pre class="editor-colors lang-text"><code>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>/&nbsp;#&nbsp;swapoff&nbsp;-a&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>swapoff:&nbsp;/dev/sda2:&nbsp;Operation&nbsp;not&nbsp;permitted&lt;/span>&lt;/span>&lt;/span>&lt;/div></code></pre>

You can create custom Seccomp profiles for your container, disabling for example calls to <a href="https://github.com/docker/labs/blob/master/security/seccomp/README.md#chmod" target="_blank">chmod</a>.

Let’s retrieve the default docker Seccomp profile:

    https://raw.githubusercontent.com/moby/moby/master/profiles/seccomp/default.json

Editing the file, you will see a whitelist of syscalls (~line 52), remove *chmod*, *fchmod* and *fchmodat* from the whitelist.

Now, launch a new container using this profile and check that the restriction is enforced:

    <div class="line"><span class="syntax--text syntax--plain"><span class="syntax--meta syntax--paragraph syntax--text"><span>#&nbsp;docker&nbsp;container&nbsp;run&nbsp;--rm&nbsp;-it&nbsp;--security-opt&nbsp;seccomp=./default.json&nbsp;alpine&nbsp;sh</span></span></span></div><div class="line"><span class="syntax--text syntax--plain"><span class="syntax--meta syntax--paragraph syntax--text"><span>/&nbsp;#&nbsp;chmod&nbsp;+r&nbsp;/usr</span></span></span></div><div class="line"><span class="syntax--text syntax--plain"><span class="syntax--meta syntax--paragraph syntax--text"><span>chmod:&nbsp;/usr:&nbsp;Operation&nbsp;not&nbsp;permitted</span></span></span></div>

## Docker container breakout {#docker-container-breakout}

**Description:** The “container breakout” term is used to denote that the Docker container has bypassed isolation checks, accessing sensitive information from the host or gaining additional privileges. In order to prevent this, we want to reduce the default container privileges. For example, the Docker daemon runs as root by default, but you can create an user-level namespace or drop some of the container root capabilities.

Looking at previously <a href="https://blog.docker.com/2014/06/docker-container-breakout-proof-of-concept-exploit/" target="_blank">exploited vulnerabilities because of default Docker configuration</a>:

“The proof of concept exploit relies on a kernel capability that allows a process to open any file in the host based on its inode. On most systems, the inode of the / (root) filesystem is 2. With this information and the kernel capability it is possible to walk the host’s filesystem tree until you find the object you wish to open and then extract sensitive information like passwords.”

**Best Practices:**

*   Drop <a href="https://docs.docker.com/engine/security/security/#linux-kernel-capabilities" target="_blank">capabilities</a> (fine-grained access control beyond the root all or nothing) that are not required by your software.
    
    *   CAP_SYS_ADMIN is a <a href="https://www.slideshare.net/jpetazzo/linux-containers-lxc-docker-and-security/19-HoweverCAPSYSADMIN_is_a_big_can" target="_blank">specially nasty one</a> in terms of security, it grants a wide range of root level permissions: mounting filesystems, entering kernel namespaces, ioctl operations...

*   Create an isolated user namespace to limit the maximum privileges of the containers over the host to the equivalent of a regular user. Avoid running containers as uid 0, <a href="https://docs.docker.com/engine/security/userns-remap/#disable-namespace-remapping-for-a-container" target="_blank">if possible</a>.

*   If you need to run a privileged container, double check that it comes from a trusted source (see [Container image authenticiy][3] below)

*   Keep an eye on dangerous mountpoints from the host: the Docker socket (*/var/run/docker.sock*), */proc*, */dev*, etc. Usually, these special mounts are required to perform the container’s core functionality, make sure you understand why and how to limit the processes that can access this privileged information. Sometimes just exposing the file system with read-only privileges should be enough, don’t give write access without questioning why. In any case, Docker does <a href="https://medium.com/@nagarwal/docker-containers-filesystem-demystified-b6ed8112a04a" target="_blank">copy-on-write</a> to prevent changes in one running container to affect the base image that might be used for other container.

**Examples:**

By default, the root account of a Docker container can create device files, you may want to restrict this:

<pre class="editor-colors lang-text"><code>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>#&nbsp;sudo&nbsp;docker&nbsp;run&nbsp;--rm&nbsp;-it&nbsp;--cap-drop=MKNOD&nbsp;alpine&nbsp;sh&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>/&nbsp;#&nbsp;mknod&nbsp;/dev/random2&nbsp;c&nbsp;1&nbsp;8&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>mknod:&nbsp;/dev/random2:&nbsp;Operation&nbsp;not&nbsp;permitted&lt;/span>&lt;/span>&lt;/span>&lt;/div></code></pre>

The root account will override any file permissions by default, this is very easy to check: just create a file with an user, *chmod* it to 600 (only owner can read & write), become root and you can read it anyway.

You may want to restrict this in your containers, specially if you have backend storage mounts with sensitive user data.

    # sudo docker run --rm -it --cap-drop=DAC_OVERRIDE alpine sh

Create a random user and *cd* to it’s home. Then:

<pre class="editor-colors lang-text"><code>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>~&nbsp;$&nbsp;touch&nbsp;supersecretfile&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>~&nbsp;$&nbsp;chmod&nbsp;600&nbsp;supersecretfile&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>~&nbsp;$&nbsp;exit&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>~&nbsp;#&nbsp;cat&nbsp;/home/user/supersecretfile&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>cat:&nbsp;can&#39;t&nbsp;open&nbsp;&#39;/home/user/supersecretfile&#39;:&nbsp;Permission&nbsp;denied&lt;/span>&lt;/span>&lt;/span>&lt;/div></code></pre>

Multiple security scanners and malware tools forge their own raw network packages from scratch, you can also forbid that:

<pre class="editor-colors lang-text"><code>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>#&nbsp;docker&nbsp;run&nbsp;--cap-drop=NET_RAW&nbsp;-it&nbsp;uzyexe/nmap&nbsp;-A&nbsp;localhost&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span>&nbsp;&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>Starting&nbsp;Nmap&nbsp;7.12&nbsp;(&nbsp;&lt;/span>&lt;span class="syntax--markup syntax--underline syntax--link syntax--https syntax--hyperlink">&lt;span>https://nmap.org&lt;/span>&lt;/span>&lt;span>&nbsp;)&nbsp;at&nbsp;2017-08-16&nbsp;10:13&nbsp;GMT&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>Couldn&#39;t&nbsp;open&nbsp;a&nbsp;raw&nbsp;socket.&nbsp;Error:&nbsp;Operation&nbsp;not&nbsp;permitted&nbsp;(1)&lt;/span>&lt;/span>&lt;/span>&lt;/div></code></pre>

Check out the complete list of capabilities <a href="https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities" target="_blank">here</a> and drop anything you containerized application doesn’t need.

By default, if you create a container without namespaces, the process inside the container belongs to root from the point of view of the host.

<pre class="editor-colors lang-text"><code>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>#&nbsp;docker&nbsp;run&nbsp;-d&nbsp;-P&nbsp;nginx&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>#&nbsp;ps&nbsp;aux&nbsp;|&nbsp;grep&nbsp;nginx&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;18951&nbsp;&nbsp;0.2&nbsp;&nbsp;0.0&nbsp;&nbsp;32416&nbsp;&nbsp;4928&nbsp;?&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Ss&nbsp;&nbsp;&nbsp;12:31&nbsp;&nbsp;&nbsp;0:00&nbsp;nginx:&nbsp;master&nbsp;process&nbsp;nginx&nbsp;-g&nbsp;daemon&nbsp;off;&lt;/span>&lt;/span>&lt;/span>&lt;/div></code></pre>

But we can create a completely separated <a href="https://docs.docker.com/engine/security/userns-remap/" target="_blank">user namespace</a>. Edit the */etc/docker/daemon.json* file and add the conf key (be careful not to break json format):

    "userns-remap": "default"

Restart the Docker daemon. This will create a preconfigured *dockremap* user. You will notice this new namespace is empty.

<pre class="editor-colors lang-text"><code>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>#&nbsp;systemctl&nbsp;restart&nbsp;docker&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>#&nbsp;docker&nbsp;ps&lt;/span>&lt;/span>&lt;/span>&lt;/div></code></pre>

Launch the nginx image again:

<pre class="editor-colors lang-text"><code>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>#&nbsp;docker&nbsp;run&nbsp;-d&nbsp;-P&nbsp;nginx&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>#&nbsp;ps&nbsp;aux&nbsp;|&nbsp;grep&nbsp;nginx&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>165536&nbsp;&nbsp;&nbsp;19906&nbsp;&nbsp;0.2&nbsp;&nbsp;0.0&nbsp;&nbsp;32416&nbsp;&nbsp;5092&nbsp;?&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Ss&nbsp;&nbsp;&nbsp;12:39&nbsp;&nbsp;&nbsp;0:00&nbsp;nginx:&nbsp;master&nbsp;process&nbsp;nginx&nbsp;-g&nbsp;daemon&nbsp;off;&lt;/span>&lt;/span>&lt;/span>&lt;/div></code></pre>

Now the *nginx* process runs in a different user namespace, increasing the isolation of the containers.

## Container image authenticity {#container-image-authenticity}

**Description:** There are plenty of Docker images and repositories on the Internet doing all kinds of awesome and useful stuff, but if you are pulling images without using any trust and authenticity mechanism, you are basically running <a href="https://blog.acolyer.org/2017/04/03/a-study-of-security-vulnerabilities-on-docker-hub/" target="_blank">arbitrary software</a> on your systems.

*   Where did the image come from?
*   Do you trust the image creator? Which security policies are they using?
*   Do you have objective cryptographic proof that the author is actually that person?
*   How do you know nobody has been tampering with the image after you pulled it?

Docker will let you pull and run anything you throw at it by default, so *encapsulation* won't save you from this. Even if you only consume your own custom images, you want to make sure nobody inside the organization is able to tamper with an image. The solution usually boils down to the classical PKI-based chain of trust.

**Best Practices:**

*   The regular Internet common sense: do not run unverified software and / or from sources you don’t explicitly trust.

*   Deploy a container-centric trust server using some of the Docker registry servers available in our <a href="https://sysdigrp2rs.wpengine.com/blog/20-docker-security-tools/" target="_blank">Docker Security Tools</a> list.

*   Enforce mandatory signature verification for any image that is going to be pulled or running on your systems.

**Example:**

Deploying a full-blown trust server is beyond the scope of this article, but you can start signing your images right away.

Get a <a href="https://hub.docker.com/" target="_blank">Docker Hub</a> account if you don’t have one already.

Create a directory containing the following trivial Dockerfile:

<pre class="editor-colors lang-text"><code>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>#&nbsp;cat&nbsp;Dockerfile&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>FROM&nbsp;alpine:latest&lt;/span>&lt;/span>&lt;/span>&lt;/div></code></pre>

Build the image:

    # docker build -t <youruser>/alpineunsigned .

Log into your Docker Hub account and submit the image:

<pre class="editor-colors lang-text"><code>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>#&nbsp;docker&nbsp;login&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>[…]&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>#&nbsp;docker&nbsp;push&nbsp;&lt;youruser&gt;/alpineunsigned:latest&lt;/span>&lt;/span>&lt;/span>&lt;/div></code></pre>

Enable Docker trust enforcement:

    # export DOCKER_CONTENT_TRUST=1

Now try to retrieve the image you just uploaded:

    # docker pull <youruser>/alpineunsigned

You should receive the following error message:

<pre class="editor-colors lang-text"><code>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>Using&nbsp;default&nbsp;tag:&nbsp;latest&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>Error:&nbsp;remote&nbsp;trust&nbsp;data&nbsp;does&nbsp;not&nbsp;exist&nbsp;for&nbsp;docker.io/&lt;youruser&gt;/alpineunsigned:&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>notary.docker.io&nbsp;does&nbsp;not&nbsp;have&nbsp;trust&nbsp;data&nbsp;for&nbsp;docker.io/&lt;youruser&gt;/alpineunsigned&lt;/span>&lt;/span>&lt;/span>&lt;/div></code></pre>

Now that DOCKER_CONTENT_TRUST is enabled, you can build the container again and it will be signed by default.

    # docker build --disable-content-trust=false -t <youruser>/alpinesigned:latest .

Now you should be able to push and pull the signed container without any security warning. The first time you push a trusted image, Docker will <a href="https://docs.docker.com/engine/security/trust/content_trust/#push-trusted-content" target="_blank">create a root key for you</a>, you will also need a repository key for the image, both will prompt for a user defined password.

Your private keys are in the *~/.docker/trust* directory, safeguard and backup them.

The DOCKER_CONTENT_TRUST is just an environment variable, and will die with your shell session but trust validation should be implemented across the entire process, from the images building, the images hosting in the registry through images execution in the nodes.

## Container resource abuse {#container-resource-abuse}

**Description:** Containers are much more numerous than virtual machines on average, they are lightweight and you can spawn big clusters of them on modest hardware. That’s definitely an advantage, but it implies that a lot of software entities are competing for the host resources. Software bugs, design miscalculations or a deliberate malware attack can easily cause a *Denial of Service* if you don’t properly configure resource limits.

To add up to the problem, there are several different resources to safeguard: CPU, main memory, storage capacity, network bandwidth, I/O bandwidth, swapping… there are <a href="https://sysdigrp2rs.wpengine.com/blog/container-isolation-gone-wrong/" target="_blank">some kernel resources that are not so evident</a>, even more obscure resources such as user IDs (UIDs) exist!.

**Best Practices:** Limits on these resources are disabled by default on most containerization systems, configuring them before deploying to production is basically a must. Three fundamental steps:

*   Use the resource limitation features bundled with the Linux kernel and/or the containerization solution.

*   Try to replicate the production loads on pre-production. Some people uses synthetic stress test, others choose to ‘replay’ the actual real-time production traffic. Load testing is vital to know where are the physical limits and where is your *normal* range of operations.

*   Implement <a href="https://sysdigrp2rs.wpengine.com/" target="_blank">Docker monitoring and alerting</a>. You don’t want to hit the wall if there is a resource abuse problem, malicious or not, you need to set thresholds and be warned before it’s too late.

**Examples:**

Control groups or *cgroups* are a feature of the Linux kernel that allow you to limit the access processes and containers have to system resources. We can configure some limits directly from the Docker command line:

    # docker run -it --memory=2G --memory-swap=3G ubuntu bash

This will limit the container to 2GB main memory, 3GB total (main + swap). To check that this is working, we can run a load simulator, for example the *stress* program present in the ubuntu repositories:

    root@e05a311b401e:/# stress -m 4 --vm-bytes 8G

You will see a ‘FAILED’ notification from the *stress* output.

If you tail the *syslog* on the hosting machine, you will be able to read something similar to:

<pre class="editor-colors lang-text"><code>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>Aug&nbsp;15&nbsp;12:09:03&nbsp;host&nbsp;kernel:&nbsp;[1340695.340552]&nbsp;Memory&nbsp;cgroup&nbsp;out&nbsp;of&nbsp;memory:&nbsp;Kill&nbsp;process&nbsp;22607&nbsp;(stress)&nbsp;score&nbsp;210&nbsp;or&nbsp;sacrifice&nbsp;child&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>Aug&nbsp;15&nbsp;12:09:03&nbsp;host&nbsp;kernel:&nbsp;[1340695.340556]&nbsp;Killed&nbsp;process&nbsp;22607&nbsp;(stress)&nbsp;total-vm:8396092kB,&nbsp;anon-rss:363184kB,&nbsp;file-rss:176kB,&nbsp;shmem-rss:0kB&lt;/span>&lt;/span>&lt;/span>&lt;/div></code></pre>

Using `docker stats` you can check current memory usage & limits. If you are using Kubernetes, on each pod definition you can actually book the resources that you application need to run properly and also define maximum limits, using <a href="https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/" target="_blank">requests and limits</a>:

<pre class="editor-colors lang-text"><code>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>[...]&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span>&nbsp;&nbsp;&lt;/span>&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>-&nbsp;name:&nbsp;wp&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span>&nbsp;&nbsp;&nbsp;&nbsp;&lt;/span>&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>image:&nbsp;wordpress&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>&nbsp;&nbsp;&nbsp;&nbsp;resources:&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;/span>&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>requests:&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;/span>&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>memory:&nbsp;"64Mi"&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cpu:&nbsp;"250m"&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;/span>&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>limits:&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;/span>&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>memory:&nbsp;"128Mi"&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cpu:&nbsp;"500m"&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>[...]&lt;/span>&lt;/span>&lt;/span>&lt;/div></code></pre>

## Docker security vulnerabilities present in the static image {#docker-security-vulnerabilities-present-in-the-static-image}

**Description:** Containers are isolated black boxes, if they are doing their work as expected it’s easy to forget which software and version is specifically running inside. Maybe a container is performing like a charm from the operational point of view but it’s running version X.Y.Z of the web server which happens to suffer from a critical security flaw. This flaw has been fixed long ago upstream, but not in your local image. This kind of problem can go unnoticed for a long time if you don’t take the appropriate measures.

**Best Practices:** Picturing the containers as immutable atomic units is really nice for architecture design, from the security perspective however, you need to regularly inspect their contents:

*   Update and rebuild you images periodically to grab the newest security patches, of course you will also need a pre-production testbench to make sure these updates are not breaking production.
    
    *   Live-patching containers is usually considered a bad practice, the pattern is to rebuild the entire image with each update. Docker has declarative, efficient, easy to understand build systems, so this is easier than it may sound at first.
    *   Use software from a distributor that guarantees security updates, anything you install manually out of the distro, you have to manage security patching yourself.
    *   Docker and microservice based approaches consider <a href="https://kubernetes.io/docs/tasks/run-application/rolling-update-replication-controller/" target="_blank">progressively rolling over updates</a> without disrupting uptime a fundamental requisite of their model.
    *   User data is clearly separated from the images, making this whole process safer.

*   Keep it simple. Minimal systems expect less frequent updates. Remember the intro, less software and moving parts equals less attack surface and updating headaches. Try to split your containers if they get too complex.

*   Use a vulnerability scanner, there are plenty out there, both free and commercial. Try to stay up to date on the security issues of the software you use subscribing to the mailing lists, alert services, etc.
    
    *   Integrate this vulnerability scanner as a mandatory step of your CI/CD, automate where possible, don’t just manually check the images now and then.

**Example:**

The are multiple Docker images registry services that offer image scanning, for this example we decided to use <a href="https://quay.io/" target="_blank">CoreOS Quay</a> that uses the open source Docker security image scanner <a href="https://github.com/coreos/clair" target="_blank">Clair</a>. Quay it’s a commercial platform but some services are free to use. You can create a personal trial account following <a href="https://quay.io/plans/" target="_blank">these instructions</a>.

Once you have your account, go to *Account Settings* and set a new password (you need this to create repos).

Click on the **+** symbol on your top right and create a new public repo:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/docker_7_security_threats.png" alt="Docker security quay account" width="1493" height="1129" class="alignleft size-full wp-image-4508" />][8] 
We go for an empty repository here, but you have several other options as you can see in the image above.

Now, from the command line, we log into the Quay registry and push a local image:

<pre class="editor-colors lang-text"><code>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>#&nbsp;docker&nbsp;login&nbsp;quay.io&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>#&nbsp;docker&nbsp;push&nbsp;quay.io/&lt;your_quay_user&gt;/&lt;your_quay_image&gt;:&lt;tag&gt;&lt;/span>&lt;/span>&lt;/span>&lt;/div></code></pre>

Once the image is uploaded into the repo you can click on it’s ID and inspect the image security scan, ordered by severity, with the associated CVE report link and also upstream patched package versions.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/cve.png" alt="Docker security vulnerability list" width="2488" height="1018" class="alignleft size-full wp-image-4507" />][9] 
## Docker credentials and secrets {#docker-credentials-and-secrets}

**Description:** Your software needs sensitive information to run: user password hashes, server-side certificates, encryption keys… etc. This situation is made worse by the containers nature: you don’t just ‘setup a server’, the microservices are plenty and may be constantly created and destroyed. You need an automatic and secure process to share this sensitive info.

**Best Practices:**

*   Do not use environment variables for secrets, this is a very common yet very unsecure practice.

*   Do not embed any secrets in the container image. Read this IBM <a href="https://wycd.net/posts/2017-02-21-ibm-whole-cluster-privilege-escalation-disclosure.html" target="_blank">post-mortem report</a>: “The private key and the certificate were mistakenly left inside the container image.”

*   Deploy a Docker credentials management software if your deployments get complex enough, we have reviewed several free and commercial options in our <a href="https://sysdigrp2rs.wpengine.com/blog/20-docker-security-tools/" target="_blank">Docker security tools</a> list. Do not attempt to create your own ‘secrets storage’ (curl-ing from a secrets server, mounting volumes, etc, etc) unless you know really really well what you are doing.

**Examples:**

First, let’s see how to capture an environment variable:

<pre class="editor-colors lang-text"><code>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>#&nbsp;docker&nbsp;run&nbsp;-it&nbsp;-e&nbsp;password=&#39;S3cr3tp4ssw0rd&#39;&nbsp;alpine&nbsp;sh&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>/&nbsp;#&nbsp;env&nbsp;|&nbsp;grep&nbsp;pass&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>password=S3cr3tp4ssw0rd&lt;/span>&lt;/span>&lt;/span>&lt;/div></code></pre>

Is that simple, even if you *su* to a regular user:

<pre class="editor-colors lang-text"><code>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>/&nbsp;#&nbsp;su&nbsp;user&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>/&nbsp;$&nbsp;env&nbsp;|&nbsp;grep&nbsp;pass&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>password=S3cr3tp4ssw0rd&lt;/span>&lt;/span>&lt;/span>&lt;/div></code></pre>

Nowadays, container orchestration systems offer some basic secret management. For example Kubernetes has the secrets resource. Docker Swarm has also its own secrets feature, that will be quickly demonstrated here:

Initialize a new Docker Swarm (you may want to do this on a VM):

    # docker swarm init --advertise-addr <your_advertise_addr>

Create a file with some random text, your secret:

<pre class="editor-colors lang-text"><code>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>#&nbsp;cat&nbsp;secret.txt&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>This&nbsp;is&nbsp;my&nbsp;secret&lt;/span>&lt;/span>&lt;/span>&lt;/div></code></pre>

Create a new secret resource from this file:

    # docker secret create somesecret secret.txt

Create a Docker Swarm service with access to this secret, you can modify the *uid*, *gid*, *mode*, etc:

    # docker service create --name nginx --secret source=somesecret,target=somesecret,mode=0400 nginx

Log into the nginx container, you will be able to use the secret:

    <div class="line"><span class="syntax--text syntax--plain"><span class="syntax--meta syntax--paragraph syntax--text"><span>root@3989dd5f7426:/#&nbsp;cat&nbsp;/run/secrets/somesecret</span></span></span></div><div class="line"><span class="syntax--text syntax--plain"><span class="syntax--meta syntax--paragraph syntax--text"><span>This&nbsp;is&nbsp;my&nbsp;secret</span></span></span></div><div class="line"><span class="syntax--text syntax--plain"><span class="syntax--meta syntax--paragraph syntax--text"><span>root@3989dd5f7426:/#&nbsp;ls&nbsp;/run/secrets/somesecret</span></span></span></div><div class="line"><span class="syntax--text syntax--plain"><span class="syntax--meta syntax--paragraph syntax--text"><span>-r--------&nbsp;1&nbsp;root&nbsp;root&nbsp;19&nbsp;Aug&nbsp;28&nbsp;16:45&nbsp;/run/secrets/somesecret</span></span></span></div>

This is a minimal proof of concept, at the very least now your secrets are properly stored and can be revoked or rotated from a central point of authority.

## Docker runtime security monitoring {#docker-runtime-security-monitoring}

**Description:** In the former sections, we have covered the *static* aspect of Docker security: vulnerable kernels, unreliable base images, capabilities that are granted or denied at launch-time, etc. But what if, despite all these, the image has been compromised during runtime and starts to show suspicious activity?

Best Practices:

*   All the previously described static countermeasures does not cover all attack vectors. What if your own in-house application has a vulnerability? Or attackers are using a 0-day not detected by the scanning? Runtime security can be compared to Windows anti-virus scanning: detect and prevent an existing break from further penetration.

*   Do not use runtime protection as a replacement for any other static up-front security practices: Attack prevention is always preferable to attack detection. Use it as an extra layer of peace-of-mind.

*   Having generous logs and events from your services and hosts, correctly stored and easily searchable and correlated with any change you do will help a lot when you have to do a post-mortem analysis.

**Example:**

<a href="https://www.sysdig.org/falco/" target="_blank">Sysdig Falco</a> is an open source, behavioral monitoring software designed to detect anomalous activity. Sysdig Falco works as an intrusion detection system on any Linux host, although it is particularly useful when using Docker since it supports container-specific context like container.id, container.image, Kubernetes resources or namespaces for its rules.

Falco rules can trigger notifications on multiple anomalous activity, let’s show a simple example of someone running an interactive shell in one of the production containers.

First, we will install Falco using the automatic installation script (not recommend for production environments, you may want to use a VM for this):

<pre class="editor-colors lang-text"><code>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>#&nbsp;curl&nbsp;-s&nbsp;&lt;/span>&lt;span class="syntax--markup syntax--underline syntax--link syntax--https syntax--hyperlink">&lt;span>https://s3.amazonaws.com/download.draios.com/stable/install-falco&lt;/span>&lt;/span>&lt;span>&nbsp;|&nbsp;sudo&nbsp;bash&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>#&nbsp;service&nbsp;falco&nbsp;start&lt;/span>&lt;/span>&lt;/span>&lt;/div></code></pre>

And then we will run an interactive shell in a nginx container:

<pre class="editor-colors lang-text"><code>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>#&nbsp;docker&nbsp;run&nbsp;-d&nbsp;--name&nbsp;nginx&nbsp;nginx&lt;/span>&lt;/span>&lt;/span>&lt;/div>&lt;div class="line">&lt;span class="syntax--text syntax--plain">&lt;span class="syntax--meta syntax--paragraph syntax--text">&lt;span>#&nbsp;docker&nbsp;exec&nbsp;-it&nbsp;nginx&nbsp;bash&lt;/span>&lt;/span>&lt;/span>&lt;/div></code></pre>

On the hosting machine tail the */var/log/syslog* file and you will be able to read:

    Aug 15 21:25:31 host falco: 21:25:31.159081055: Debug Shell spawned by untrusted binary (user=root shell=sh parent=anacron cmdline=sh -c run-parts --report /etc/cron.weekly pcmdline=anacron -dsq)

Sysdig Falco doesn’t need to modify or instrument containers in any way. This is just a trivial example of Falco capabilities, <a href="https://github.com/draios/falco/wiki/Falco-Examples" target="_blank">check out these examples</a> to learn more.

## Conclusions {#conclusions}

Docker itself is build with security in mind and some of their inherent features can help you protecting your system. Don’t get too confident though, there is no magic bullet here other than keeping up with the state-of-the-art Docker security practices but there are a few container-specific security tools that can help giving battle to all these Docker vulnerabilities and threats.

Hopefully these simple examples have stirred up your interest in the matter!

 [1]: #docker-host-and-kernel-security
 [2]: #docker-container-breakout
 [3]: #container-image-authenticity
 [4]: #container-resource-abuse
 [5]: #docker-security-vulnerabilities-present-in-the-static-image
 [6]: #docker-credentials-and-secrets
 [7]: #docker-runtime-security-monitoring
 [8]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/docker_7_security_threats.png
 [9]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2017/08/cve.png