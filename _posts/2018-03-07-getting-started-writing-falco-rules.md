---
ID: 6661
post_title: Getting Started Writing Falco Rules
author: Michael Ducy
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/getting-started-writing-falco-rules/
published: true
post_date: 2018-03-07 06:30:41
---
﻿Sysdig's <a href="https://www.sysdigrp2rs.wpengine.com/opensource/falco" target="_blank">Falco</a> is a powerful behavioral activity monitoring tool to detect abnormal behavior in your applications and containers. While it comes with 25 rules for common best practices, you’ll quickly want to start writing custom rules to match your operational and application requirements. Impatient and don’t want to wait? You can learn how to install Falco over on the <a href="https://github.com/draios/falco/wiki" target="_blank">project’s GitHub wiki</a>.

## Sysdig Filter Syntax {#sysdigfiltersyntax}

To get started writing a rule, it’s important to understand the Sysdig filter syntax you’ll use to write the rule conditions. Sysdig filters expose a variety of information about system calls and events that take place on a system. These filters are organized into a set of classes, called “Field Classes”. If you have Sysdig installed, you can always query the latest classes by running `sysdig -l`. A list of the classes & fields can also be found in <a href="https://www.sysdig.org/wiki/sysdig-user-guide/#user-content-filtering" target="_blank">the documentation</a>. Below are a list of the classes Sysdig offers. You can see that there is a rich library of classes (and fields) available.

**fd** - File Descriptors  
**process** - Processes  
**evt** - System Events  
**user** - Users  
**group** - Groups  
**syslog** - Syslog messages  
**container** - Container info  
**fdlist** - FD poll events  
**k8s** - Kubernetes events  
**mesos** - Mesos events  
**span** - Start/Stop markers  
**evtin** - Filter based on Spans

## Rule Basics {#rulebasics}

Falco rules are written in YAML, and have a variety of required and optional keys.

**rule:** Name of the rule.  
**desc:** Description of what the rule is filtering for.  
**condition:** The logic statement that triggers a notification.  
**output:** The message that will be shown in the notification.  
**priority:** The “logging level” of the notification.  
**tags:** Used to categorize rules. (optional)  
**enabled:** Turn the rule on or off (optional, defaults to true)

Using the required keys, we can write a very basic rule:

    - rule: Detect bash in a container
      desc: You shouldn’t have a shell run in a container
      condition: container.id != host and proc.name = bash
      output: Bash ran inside a container (user=%user.name command=%proc.cmdline %container.info)
      priority: INFO

While this is a useful rule, systems provide more shells than just bash. Falco allows you to define \`lists\` that contain a set of items. We can use this to tell Falco to alert whenever a shell from the list is executed inside a container.

    - list: system_shells
      items: [bash, zsh, ksh, sh, csh]

With this list defined, we can then modify our rule to read from the list for the process name.

    - rule: Detect shells in a container
      desc: You shouldn’t have a shell ran in a container
      condition: container.id != host and proc.name in (system_shells)
      output: Bash ran inside a container (user=%user.name command=%proc.cmdline %container.info)
      priority: INFO

Falco will now alert you whenever one of these 5 shells are executed inside a container. Of course, Falco provides a more complete rule for this use case in it's default rule set. 

## Learning More {#learningmore}

As we’ve seen, Falco rules are simple to write, but provide a very powerful tool in detecting abnormal system behavior. In the world of containers, immutable infrastructure, and microservices, Falco can help ensure that users are following best practices of these new paradigms. Falco rules can also help alert you to systems that might be compromised, by alerting on abnormal binaries being executed (e.g. cryptomining) or abnormal network connections.

Need more info? Check out the <a href="https://github.com/draios/falco/wiki" target="_blank">Falco documentation</a> on the GitHub wiki, or join the conversation on the <a href="https://slack.sysdigrp2rs.wpengine.com/" target="_blank">Sysdig Slack team</a>.

## Sysdig Secure {#sysdigsecure}

If you like the functionality of Sysdig Falco, but want more integrations, user activity auditing, the ability to kill or pause containers based on rules, and the ability to capture the state of a container when a rule fails check out Sysdig Secure. You can request a <a href="https://go.sysdigrp2rs.wpengine.com/docker-security-demo" target="_blank">customized demo</a>, or <a href="https://go.sysdigrp2rs.wpengine.com/secure-trial-request" target="_blank">kick off a trial</a>.