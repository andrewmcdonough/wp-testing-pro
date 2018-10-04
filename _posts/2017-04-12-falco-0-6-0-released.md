---
ID: 3991
post_title: Falco 0.6.0 Released
author: Mark Stemm
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/falco-0-6-0-released/
published: true
post_date: 2017-04-12 18:18:14
---
We just released Falco 0.6.0. This has several great new features as well as continued improvement to the default ruleset. Here's a summary of the changes:

## Tags for Falco Rules {#tagsforfalcorules}

Rules now have an optional set of *tags* that are used to categorize the ruleset into groups of related rules. Here's an example:

    - rule: File Open by Privileged Container
      desc: Any open by a privileged container. Exceptions are made for known trusted images.
      condition: (open_read or open_write) and container and container.privileged=true and not trusted_containers
      output: File opened for read/write by privileged container (user=%user.name command=%proc.cmdline %container.info file=%fd.name)
      priority: WARNING
      tags: [container, cis]
    

In this case, the rule "File Open by Privileged Container" has been given the tags "container" and "cis". If the tags key is not present for a given rule or the list is empty, a rule has no tags.

Here's how you can use tags:

*   You can use the `-T <tag>` argument to disable rules having a given tag. `-T` can be specified multiple times. For example, to skip all rules with the "filesystem" and "cis" tags you would run falco with `falco -T filesystem -T cis ...`. `-T` can not be specified with `-t`.
*   You can use the `-t <tag>` argument to *only* run those rules having a given tag. `-t` can be specified multiple times. For example, to only run those rules with the "filesystem" and "cis" tags, you would run falco with `falco -t filesystem -t cis ...`. `-t` can not be specified with `-T` or `-D <pattern>` (disable rules by rule name regex).

## Tags for Current Falco Ruleset {#tagsforcurrentfalcoruleset}

We've also gone through the default ruleset and tagged all the rules with an initial set of tags. Here are the tags we've used:

*   filesystem: the rule relates to reading/writing files
*   sofware_mgmt: the rule relates to any software/package management tool like rpm, dpkg, etc.
*   process: the rule relates to starting a new process or changing the state of a current process.
*   database: the rule relates to databases
*   host: the rule *only* works outside of containers
*   shell: the rule specifically relates to starting shells
*   container: the rule *only* works inside containers
*   cis: the rule is related to the CIS Docker benchmark.
*   users: the rule relates to management of users or changing the identity of a running process.
*   network: the rule relates to network activity

Rules can have multiple tags if they relate to multiple of the above. Every rule in the falco ruleset currently has at least one tag.

## Standalone Kernel Module {#standalonekernelmodule}

Prior to 0.6.0, falco used the kernel module from sysdig called `sysdig-probe`. As of 0.6.0, falco uses its own kernel module `falco-probe`. The kernel modules are actually built from the same source code, but having a falco-specific kernel module allows falco and sysdig to be updated independently without compatibility problems.

By default, the kernel module will be installed when installing the falco debian/redhat package or when running the `sysdig/falco` docker image. The script that installs the kernel module tries to install it in 3 different ways:

*   Build the kernel module from source using [dkms][1].
*   Download a pre-built kernel module from downloads.draios.com.
*   Look for a pre-built kernel module from `~/.sysdig`.

For options using a pre-built kernel module, the kernel module should have the following filename: `falco-probe-<falco version>-<arch>-<kernel release>-<kernel config hash>.ko` `<kernel config hash>` is a md5sum of the config file that sets kernel options (e.g. `/boot/config-4.4.0-64-generic`). This file can reside in other locations--see the [kernel module builder script][2] for full details on the set of paths it tries to find the kernel config file.

## Other New Features {#othernewfeatures}

This release also includes some other new features:

*   When specifying multiple rule sets by specifying `-r` multiple times, you can override rules/macros/lists from earlier files in later files.
*   We added example k8s yaml files that show how to run falco as a k8s DaemonSet and how to run falco-event-generator as a deployment running on one node.
*   Falco now compiles on OSX. Like sysdig, on OSX it is limited to reading trace files.
*   We've run the falco docker image through docker hub's [security scanner][3] and updated third-party libraries to address security vulnerabilities.

## Rule Changes {#rulechanges}

This release has a ton of rule changes to eliminate false positives. The full list is too long to include here, but here are some examples where we've improved coverage:

*   When running the k8s liveness checking utility [exechealthz][4].
*   When managing software using [ansible][5] or monitoring software via [npre][6].
*   When running other security products such as [aide][7], [bro][8], and [icinga2][9].

## Learn More {#learnmore}

For the full set of changes in this release, you can always look at the [changelog at github][10].

The release is available via the usual channels–rpm/debian packages, [docker hub][11] and [github][12].

Let us know if you have any issues, and enjoy!

 [1]: https://en.wikipedia.org/wiki/Dynamic_Kernel_Module_Support
 [2]: https://github.com/draios/sysdig/blob/dev/scripts/sysdig-probe-loader
 [3]: https://docs.docker.com/docker-cloud/builds/image-scan/
 [4]: https://github.com/kubernetes/contrib/tree/master/exec-healthz
 [5]: https://github.com/ansible/ansible
 [6]: https://exchange.nagios.org/directory/Addons/Monitoring-Agents/NRPE--2D-Nagios-Remote-Plugin-Executor/details
 [7]: http://aide.sourceforge.net/
 [8]: https://github.com/bro
 [9]: https://github.com/Icinga/icinga2
 [10]: https://github.com/draios/falco/blob/dev/CHANGELOG.md
 [11]: https://hub.docker.com/r/sysdig/falco
 [12]: https://github.com/draios/falco