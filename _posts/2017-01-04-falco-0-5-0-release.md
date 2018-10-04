---
ID: 3559
post_title: Falco 0.5.0 now available
author: Mark Stemm
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/falco-0-5-0-release/
published: true
post_date: 2017-01-04 22:04:09
---
## Falco 0.5.0 Released

We recently released Falco 0.5.0, the behavioral security monitor. This release has a little bit of everything--new features, rule changes, and bug fixes. Here's a rundown of the changes:

## New Security Monitoring Features {#newsecuritymonitoringfeatures}

Usually, you'll want your ruleset to result in few-to-no falco notifications. However, it's possible that the ruleset could be noisy in new environments. We made a couple of changes to mitigate the impact of a flood of notifications:

*   Falco now caches output formatting objects so they are not recreated for every notification.
*   Notifications are now throttled by a token bucket that enforces an average rate (notifications/second) as well as allowing for temporary bursts above the average rate.

There are some other new features:

*   Falco can now record statistics on event processing to a file, via the `-s <statistics file>` option. This allows you to monitor the stream of events from the kernel module to ensure none are dropped.
*   In some cases, you may want to run Falco in a container without explicitly building/loading the kernel module. You can use the `SYSDIG_SKIP_LOAD` environment variable in the `docker run` command will skip those steps and assume the driver is already loaded.
*   The verbosity of Falco's log messages can now be controlled via the `log_levels` configuration option.

## Falco Rule Changes {#falcorulechanges}

The ruleset has several changes to reduce noisiness:

*   dnf has been added to the list of package management programs.
*   fail2ban-server and apt/apt-get have been added as programs that can spawn shells.
*   systemd has been added as a program that can access sensitive files.
*   google_containers/kube-proxy has been added as a trusted image.

## Bug Fixes {#bugfixes}

There are also a few bug fixes:

*   Prior to 0.5.0, if your rule had a malformed `output` field, you wouldn't know about it until a rule actually triggered and tried to use the field. Now, all output fields are validated at load time.
*   If you were building from source and tried to provide your own third-party libraries, you'd see that the `USE_BUNDLED_DEPS` cmake option wasn't working. This has been fixed.
*   We updated the download links in our installer scripts to solely use https.

For the full set of changes in this release, you can always look at the [changelog at github][1].

The release is available via the usual channels–rpm/debian packages, [docker hub][2] and [github][3].

Let us know if you have any issues, and enjoy!

 [1]: https://github.com/draios/falco/blob/dev/CHANGELOG.md
 [2]: https://hub.docker.com/r/sysdig/falco
 [3]: https://github.com/draios/falco