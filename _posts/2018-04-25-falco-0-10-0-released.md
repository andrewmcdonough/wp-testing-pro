---
ID: 7133
post_title: Falco 0.10.0 released.
author: Mark Stemm
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/falco-0-10-0-released/
published: true
post_date: 2018-04-25 16:46:21
---
We are happy to announce the [release of Falco 0.10.0][1]. This release incorporates a number of improvements focused on making Falco easier to deploy, improvements with rules, and improvements in the system call events Falco supports. 

## Deployment Improvements

### Rules Directory Support

With this release, you can specify a rules directory and Falco will read all rules files in that directory, loading them in alphabetical order. The default packaging includes an (empty) rules directory in `/etc/falco/rules.d`. In future releases, we plan on providing specific rulesets that can be copied/symlinked into `/etc/falco/rules.d`.

### Sample Puppet Module

We've added a [sample Puppet Module][2] to manage Falco. This module configures the main Falco configuration file `/etc/falco/falco.yaml`, providing templates for all configuration options. It installs Falco using Debian/rpm packages and installs/manages it as a systemd service.

We've also pushed this module to [Puppet Forge][3].

### Log Rotation Support

If you'd like to set up automatic log rotation for Falco, we've included an example [logrotate config][4] and changed Falco to close/reopen its log files when signalled with USR1.

## Rule Improvements

Like every release, we've updated the set of rules to improve coverage and reduce false positives based on feedback from the community. In addition, there are a few new rules:

*   `Disallowed SSH Connection`<span> </span>detects ssh connection attempts to hosts outside of an expected set. In order to be effective, you need to override the macro<span> </span>`allowed_ssh_hosts`<span> </span>in a user rules file. 
*   `Unexpected K8s NodePort Connection`<span> </span>detects attempts to contact the Kubernetes NodePort range from a program running inside a container. In order to be effective, you need to override the macro<span> </span>`nodeport_containers`<span> </span>in a user rules file.
*   `Unexpected UDP Traffic`<span> </span>checks for udp traffic not on a list of expected ports. It's somewhat FP-prone, so it must be explicitly enabled by overriding the macro<span> </span>`do_unexpected_udp_check`<span> </span>in a user rules file.

## Updated Support for Syscalls

We've added support for all syscalls supported by Sysdig, including those that do not have automatic parameter extraction by the kernel module. Previously, this had limited support but might not have worked in all cases. In addition, to improve resource usage, we further restricted the set of system calls available to Falco. You can continue to have access to all system calls with `-A`, and you can now see the specific set of skipped system calls with `-i`.

## Cryptojacking Example

We've added an [example][5] of how an overly permissive Docker configuration can be exploited by malicious cryptojacking software and how Falco detects the attack. This is related to our [blog post][6] from January, which discussed the same topic in the context of Kubernetes.

## Other Changes

Some other improvements include:

*   Allowing validation of multiple rules files on one command line with `-V`.
*   More compact json output that skips the preformatted `output` field.
*   Add the ability to suppress warnings about event type use on a rule-by-rule basis.
*   Fix `fd.net` so it can work with groups of netmasks e.g. `evt.type=connect and fd.net in ("127.0.0.1/24")`.
*   Fixing use of keep-alive when using both program and file outputs.
*   Fixing bugs related to rule order and skipped rules.

## Further Information

For the full set of changes in this release, please look at the release's [changelog on github][7].The release is available via the usual channels–rpm/Debian packages, [Falco Docker images][8] and [GitHub][9].

Let us know if you have any issues over in the Sysdig open source [Slack team][10], and enjoy!

 [1]: https://github.com/draios/falco/releases/tag/0.10.0
 [2]: https://github.com/draios/falco/tree/dev/examples/puppet-module/sysdig-falco
 [3]: https://forge.puppet.com/sysdig/falco
 [4]: https://github.com/draios/falco/blob/dev/examples/logrotate/falco
 [5]: https://github.com/draios/falco/tree/dev/examples/bad-mount-cryptomining
 [6]: https://sysdigrp2rs.wpengine.com.com/blog/detecting-cryptojacking/
 [7]: https://github.com/draios/falco/blob/dev/CHANGELOG.md
 [8]: https://hub.docker.com/r/sysdig/falco
 [9]: https://github.com/draios/falco
 [10]: https://slack.sysdigrp2rs.wpengine.com/