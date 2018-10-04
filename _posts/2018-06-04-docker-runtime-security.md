---
ID: 7416
post_title: >
  Implementing Docker/Kubernetes runtime
  security.
author: Mateo Burillo
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/docker-runtime-security/
published: true
post_date: 2018-06-04 06:18:25
---
<a href="https://sysdigrp2rs.wpengine.com/opensource/falco/" target="_blank">Sysdig Falco</a> performs live monitoring of the behavior of your Docker containers and Kubernetes pods at runtime. Using Falco you can create a Docker runtime security policy to detect attacks and anomalous activity on production environments, in real-time, so you can react to unknown and 0-day vulnerabilities, attacks caused by weak or leaked credentials or compliance breaches.

Sysdig Falco has <a href="https://sysdigrp2rs.wpengine.com/blog/getting-started-writing-falco-rules/" target="_blank">its own security policy rules syntax</a>, based on Sysdig filtering language which is very similar to tcpdump syntax, self-descriptive and fast to learn. In addition, it is really easy to install Falco in any container orchestration platform. If you are looking for a commercial product instead, check out <a href="https://sysdigrp2rs.wpengine.com/product/secure/" target="_blank">Sysdig Secure</a>, built on top of Sysdig Falco open-source technology. [tweet_box design="default" float="none"]Looking at implementing #Kubernetes #Docker runtime #security and you don't know where to start? With the new @sysdig #falco default ruleset is easier than ever![/tweet_box] 

Implementing runtime security policies requires to know exactly how your containers and applications behave. Since most of us build our Cloud Native applications using open-source components, we wanted to help you in this process providing base security profiles for a growing list of the most popular container images, through a <a href="https://github.com/draios/falco-extras" target="_blank">GitHub repository: falco-extras</a>, including:

*   Kubernetes 1.10 cluster components:
    *   apiserver
    *   controller-manager
    *   kube-dns
    *   kube-scheduler
    *   dashboard
*   Google Kubernetes Engine components
*   Apache
*   Consul
*   ElasticSearch
*   etcd
*   Fluentd
*   HAproxy
*   MongoDB
*   Nginx
*   PHP-FPM
*   PostgreSQL
*   Redis
*   Traefik

Previously, we did cover how to setup a few runtime security <a href="https://sysdigrp2rs.wpengine.com/blog/sysdig-secure-docker-run-time-security/" target="_blank">policy examples with Sysdig Secure</a>, now we will see how to do this with open-source.

## What can you find in a Docker runtime security policy? {#whatcanyoufindinadockerruntimesecuritypolicy}

We already showed how to implement runtime security and harden/<a href="https://sysdigrp2rs.wpengine.com/blog/kubernetes-security-harden-kube-system/" target="_blank">secure your Kubernetes kube-system namespace</a>, now we will see how to expand this to the rest of your Dockerized applications.

For each of your container images, you can create a whitelisting policy for:

*   Which processes / commands are allowed to run inside the Docker container
*   Processes that can start an outbound connection or accept an inbound connection
    *   Including allowed listening ports or remote ports
*   Files and directories that can be read
    *   By which processes
*   Files and directories that can be written
    *   By which processes
*   Even the set of Linux system calls that your container processes are expected to use!

Basically, we are creating a tightly constrained execution environment. Anything else, any unexpected behavior will raise an alarm.

For example, this is the default Docker runtime security policy for a Nginx container:

<script src="https://gist.github.com/c43b86dbba9d73ee53c566434d9bc8e5.js"></script> <noscript>
  <pre><code>
File: FalcoNginxRuleset.yaml
----------------------------

- macro: nginx_consider_syscalls
  condition: (evt.num &lt; 0)

- macro: app_nginx
  condition: container and container.image contains "nginx"

# Any outbound traffic raises a WARNING

- rule: Unauthorized process opened an outbound connection (nginx)
  desc: A nginx process tried to open an outbound connection and is not whitelisted
  condition: outbound and evt.rawres &gt;= 0 and app_nginx
  output: Non-whitelisted process opened an outbound connection (command=%proc.cmdline
    connection=%fd.name)
  priority: WARNING


# Restricting listening ports to selected set

- list: nginx_allowed_inbound_ports_tcp
  items: [80, 443, 8080, 8443]

- rule: Unexpected inbound tcp connection nginx
  desc: Detect inbound traffic to nginx using tcp on a port outside of expected set
  condition: inbound and evt.rawres &gt;= 0 and not fd.sport in (nginx_allowed_inbound_ports_tcp) and app_nginx
  output: Inbound network connection to nginx on unexpected port (command=%proc.cmdline pid=%proc.pid connection=%fd.name sport=%fd.sport user=%user.name %container.info image=%container.image)
  priority: NOTICE

# Restricting spawned processes to selected set

- list: nginx_allowed_processes
  items: ["nginx", "app-entrypoint.", "basename", "dirname", "grep", "nami", "node", "tini"]

- rule: Unexpected spawned process nginx
  desc: Detect a process started in a nginx container outside of an expected set
  condition: spawned_process and not proc.name in (nginx_allowed_processes) and app_nginx
  output: Unexpected process spawned in nginx container (command=%proc.cmdline pid=%proc.pid user=%user.name %container.info image=%container.image)
  priority: NOTICE

# Restricting files read or written to specific set

- list: nginx_allowed_file_prefixes_readwrite
  items: ["/var/log/nginx", "/var/run"]
# Remember to add your nginx cache path

- rule: Unexpected file access readwrite for nginx
  desc: Detect an attempt to access a file readwrite other than below an expected list of directories
  condition: (open_write) and not fd.name pmatch (nginx_allowed_file_prefixes_readwrite) and app_nginx
  output: Unexpected file accessed readwrite for nginx (command=%proc.cmdline pid=%proc.pid file=%fd.name %container.info image=%container.image)
  priority: NOTICE

# Restricting syscalls to selected set

- list: nginx_allowed_syscalls
  items: [accept, bind, clone, connect, dup, listen, mkdir, open, recvfrom, recvmsg, sendto, setgid, setuid, socket, socketpair]

- rule: Unexpected syscall nginx
  desc: Detect a syscall in a nginx container outside of an expected set
  condition: nginx_consider_syscalls and not evt.type in ("&lt;unknown&gt;", nginx_allowed_syscalls) and app_nginx
  output: Unexpected syscall in nginx container (command=%proc.cmdline pid=%proc.pid user=%user.name syscall=%evt.type args=%evt.args %container.info image=%container.image)
  priority: NOTICE
  warn_evttypes: False

</code></pre>
</noscript>

And of course, that's on top of many other suspicious behaviors that Falco will detect by default:

*   A shell is run inside a container
*   A container is running in privileged mode or is mounting a sensitive path like /proc from the host
*   A server process spawns a child process of an unexpected type
*   Unexpected read of a sensitive file (like /etc/shadow)
*   A non-device file is written to /dev
*   A standard system binary (like ls) makes an outbound network connection
*   And many more <a href="https://github.com/draios/falco/blob/dev/rules/falco_rules.yaml" target="_blank">policy examples</a>…

## How to customize these rules to create your runtime security profiles and policies {#howtocustomizetheserulestocreateyourruntimesecurityprofilesandpolicies}

Using these <a href="https://github.com/draios/falco-extras" target="_blank">default Falco runtime security rulesets</a> will hopefully save you a lot of time writing boilerplate. However, bear in mind that every version of a Docker container image, even for the same microservice, it's probably going to have some differences: user-defined data directories, different binary paths, scripts that need to access some external port or device, etc.

Thus, you will need to adapt the templates to your specifics before actually using them.

### How to apply runtime security policy to specific container images {#howtoapplyruntimesecuritypolicytospecificcontainerimages}

In this generic rule we just used we targeted any container image which name contains "nginx":

    - macro: app_nginx
      condition: container and container.image contains "nginx"

But you can be much more specific, using the unique hash code of the image you are deploying:

    - macro: app_nginx
      condition: container and container.image contains "73acd1f0cfadf6f56d30351ac633056a4fb50d455fd95b229f564ff0a7adecda" Falco can also leverage metadata from Kubernetes, allowing you to create more accurate and context-specific rulesets that make use of 

*Pod*, *Deployment* name or *Namespace* name. 
You have some example rules of Falco using Kubernetes metadata in <a href="https://sysdigrp2rs.wpengine.com/blog/detecting-cryptojacking-with-sysdigs-falco/" target="_blank">Detecting Cryptojacking with Sysdig's Falco</a>:

    - macro: node_app_frontend
      condition: k8s.ns.name = node-app and k8s.pod.label.role = frontend and k8s.pod.label.app = node-app
    
    - rule: Detect crypto miners using the Stratum protocol
      desc: Miners typically specify the mining pool to connect to with a URI that begins with 'stratum+tcp'
      condition: node_app_frontend and spawned_process and container.id != host and proc.cmdline contains stratum+tcp
      output: Possible miner ran inside a container (command=%proc.cmdline %container.info)
      priority: WARNING

### Detect not allowed or forbidden Docker container images {#detectnotallowedorforbiddendockercontainerimages}

Additionally, you can create your own image whitelist, to make sure no unknown or not allowed container image is ever run on your hosts / cluster:

    - list: container_image_whitelist
      items: ["sha256:8ac48589692a53a9b8c2d1ceaa6b402665aa7fe667ba51ccc03002300856d8c7", "sha256:f3fcd0775c4e15d4b73d2b62f60efb1872bf4d3328be52a0fc9a2b0951bbcc1e"]
    
    - rule: Unknown container image running
      desc: There is a container running that doesn't match any of the whitelisted sha256 codes
      condition: container and not container.image in (container_image_whitelist)
      output: Unknown container (command=%proc.cmdline pid=%proc.pid file=%fd.name %container.info image=%container.image)
      priority: WARNING

### Fine tune your runtime security: Sysdig + Sysdig Inspect (also forensics and post-mortem analysis!) {#finetuneyourruntimesecuritysysdigsysdiginspectalsoforensicsandpostmortemanalysis}

If you want a fine-grained inspection tool coupled together with a nice GUI, without leaving FOSS territory, you can always use <a href="https://github.com/draios/sysdig/wiki/Sysdig-Examples" target="_blank">sysdig</a> together with <a href="https://sysdigrp2rs.wpengine.com/blog/sysdig-inspect/" target="_blank">Sysdig Inspect</a>.

For example, if you want to record a live capture of every activity that *mynginx _container* is registering, just run:

    # sysdig -pc -s 4096 container.name=mynginx -w nginxcap.scap

After a few seconds, you can directly open the capture file on your laptop using Sysdig Inspect to browse the container activity and figure out if there is any additional constraints you can add to your runtime security profile.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/docker_runtime_security_sysdig_inspect_dashboard-1024x549.png" alt="Docker Runtime Security Sysdig Inspect Dashboard" width="820" height="440" class="alignleft size-large wp-image-7425" />][1]

From here, I can easily browse the processes that the container was running:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/docker_runtime_security_sysdig_inspect_process-1024x589.png" alt="Docker Runtime Security Sysdig Inspect Process" width="820" height="440" class="alignleft size-large wp-image-7424" />][2]

Or its network connections:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/docker_runtime_security_sysdig_inspect_network-1024x588.png" alt="Docker Runtime Security Sysdig Inspect Network" width="820" height="440" class="alignleft size-large wp-image-7423" />][3]

And much more! File system activity, system calls… everything really! That's why Sysdig Inspect is often used for troubleshooting (<a href="https://sysdigrp2rs.wpengine.com/blog/debug-kubernetes-crashloopbackoff/" target="_blank">How to troubleshoot a Kubernetes CrashLoopBackOff</a>) or forensics too.

### Help us build the Docker and Kubernetes runtime security library! {#helpusbuildthedockerandkubernetesruntimesecuritylibrary}

We would love to hear about your experience implementing runtime security in Docker and Kubernetes. Are you using Sysdig Falco already? Or wondering <a href="https://sysdigrp2rs.wpengine.com/blog/selinux-seccomp-falco-technical-discussion/" target="_blank">how it compares with seccomp or SELinux/AppArmor</a>? If you have created a set of profiles and are proud of them why not contributing back? Fork and send us Pull Requests to <a href="https://github.com/draios/falco-extras" target="_blank">falco-extras</a>!. Cannot use Falco because you are missing a specific feature or maybe you hit a bug? We also want to know! Find us on <a href="https://slack.sysdigrp2rs.wpengine.com/" target="_blank">Sysdig Slack</a>, #falco channel.

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/docker_runtime_security_sysdig_inspect_dashboard.png
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/docker_runtime_security_sysdig_inspect_process.png
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/06/docker_runtime_security_sysdig_inspect_network.png