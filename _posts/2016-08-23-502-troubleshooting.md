---
ID: 2971
post_title: 'When APM fails: A 502 troubleshooting tale'
author: Alex Diaz
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/502-troubleshooting/
published: true
post_date: 2016-08-23 08:50:49
---
Too often monitoring and performance management tools are good at giving you the “what” of your software but don’t help with the “why.” But, it’s usually the “why” that eats up hours or days of your time. Tracking down bugs and performance issues can be incredibly tricky, no matter how many good tools you have at your disposal.

This is a real customer case that we faced recently, where - get this - even our own commercial product didn’t entirely solve the mystery. Read on to hear about the case of the mysterious 502 and how open source troubleshooting tools helped a developer get on with his life.

### The environment & the problem

Our customer has a pretty standard multi-tier environment. They are deployed in AWS, so naturally are using ELB for L4 load balancing. They are using a router, HAproxy for reverse proxying across their java applications, backed by Postgres & RDS databases. All of this is containerized, managed and orchestrated by [Rancher][1].

Interestingly, the customer had three commercial tools that could help with this issue. AppDynamics was providing application-level tracing within the java application, while DataDog and [Sysdig][2] were providing infrastructure monitoring. (The customer was not insane - they were in transition between the latter two products!)

The issue itself was with a their Java application. DataDog and Sysdig were both reporting a 502 error from the HAproxy frontend. On the AppDynamics side they were seeing a Java exception.

All I had was the port number used by my HAProxy and the mysterious 502 error. It was not enough information to figure this out with my current monitoring tools: Given how AppDynamics was set up in this environment, it would have been really helpful if the java app was blowing up and we needed to trace the transaction within the application - but it wasn’t. Sysdig Cloud and DataDog were pointing me to the high level problem, but not much more. Now it gets more interesting…

Dropping in the container using the (console in rancher) and posting to the application shows success, so the problem doesn’t seem to be in the application, seems to be between the router and haproxy at the http level

<pre>$ curl -XPOST -d 'notificationAttemptId=1110769&response=0&notificationId=1036209' http://localhost:56789/cgi-bin/WebObjects/Router.woa/wa/NotificationAcknowledge/ackNotification
Success</pre>

The PHNN code in the haproxy response is mentioned [in the docs][3] as an empty, or an improperly formed HTTP header. I knew it isn’t empty because I’m getting a “success” response.

### Sysdig Captures

Well, no good news so far. Despite our wealth of tools, we still don’t have an answer. We need a quick, easy way to go deeper into our processes and understand what’s happening. And we’ve got one tool within our Sysdig toolbox that we haven’t used yet: sysdig captures.

In case you’re not familiar with sysdig captures, here’s the quick intro. Capture files are an element of the open source sysdig troubleshooting tool. They are a record of every system call that passes through your host. They are really just a flat file, with one record per line. The information is really rich, allowing you to see everything that a process is doing, including network activity, file activity, container-related activity and more. If you’re familiar with Wireshark, captures are similar in format and use, but sysdig captures are much richer.

If you want to read more about captures, read [Troubleshooting Containers after they’re long gone][4].

You can independently use sysdig captures with open source sysdig, but using Sysdig’s monitoring tool allows you to store captures centrally and even trigger captures on alerts. That’s especially useful for large environments or ones where you’re not manning the helm 24/7. In addition, in dynamic environments where containers or apps are coming and going based on an orchestration system, sysdig captures allow you to record the state of the system and troubleshoot it at a later time, even though things may have changed. It’s a bit like having video surveillance footage whenever you need it.

### The 502 error - caught in the act

Since the customer was using Sysdig’s monitoring service, I went to the sysdig cloud interface and started a capture job during the period I was receiving those HA Proxy 502 errors.

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/08/tracecapture.png"></a>[![csysdig][5]][5] 

So I got the following trace file:

<pre>$ ls
$ Capture160720094043.scap</pre>

For quite a bit of time, sysdig has offered a chisel to inspect the activity of a given set of file descriptors. It’s called *echo_fds*, and you can easily use it to troubleshoot network connections using the appropriate filter.I used this to search for network traffic that happened on port 56789 (what the java program was listening on) gives me the content of the POST and the haproxy response.

<pre>$ sysdig -r capture160720094043.scap  -s 2000 -A -c echo_fds fd.port=56789 and evt.buffer contains 502</pre>

Resulting in this:

<pre>notificationAttemptId=1111502&response=0&notificationId=1036011
------ Read 470B from   10.XXX.XXX.XXX:42080-&gt;10.XXX.XXX.XXX:56789 (java)

POST /cgi-bin/WebObjects/Router.woa/wa/NotificationAcknowledge/ackNotification HTTP/1.1
host: secure.XXXXXXXXXXXX.eu
Accept: text/html, image/gif, image/jpeg, *; q=.2, */*; q=.2
Content-type: application/x-www-form-urlencoded
User-Agent: Java/1.8.0_91
X-Forwarded-For: 52.XXX.XXX.XXX
X-Forwarded-Proto: https
Content-Length: 63
X-Forwarded-Port: 80
X-Forwarded-For: 10.XXX.XXX.XXX
Connection: close

notificationAttemptId=1106125&response=0&notificationId=1038502
------ Write 205B to   10.XX.XX.215:41590-&gt;10.XXX.XXX.XXX:56789 (haproxy)

HTTP/1.0 502 Bad Gateway
Cache-Control: no-cache
Connection: close
Content-Type: text/html

502 Bad Gateway
The server returned an invalid or incomplete response.

</pre>

So I used sysdig to see exactly what the java program was returning at the http level when I captured the trace:

<pre>$ sysdig -r capture160720094043.scap  -s 2000 -A -c echo_fds fd.port=56789 and evt.buffer contains Success | less

<span style="color: #0000ff;">------ Write 7B to 10.XXX.XXX.XXX:39814-&gt;10.XXX.XXX.XXX:56789 (java)</span>

<span style="color: #ff0000;">Success
------ Read 114B from 10.XXX.XXX.XXX:33326-&gt;10.XXX.XXX.XXX:56789 (haproxy)</span>
<span style="color: #ff0000;">
HTTP/1.0 200 Apple WebObjects
text/plain: content-type
x-webobjects-loadaverage: 0
content-length: 7</span>

<span style="color: #0000ff;">Success
------ Write 7B to 10.XXX.XXX.XXX:42904-&gt;10.XXX.XXX.XXX:56789 (java)</span>

<span style="color: #0000ff;">Success
------ Write 7B to 10.XXX.XXX.XXX:33326-&gt;10.XXX.XXX.XXX:56789 (java)</span>

<span style="color: #0000ff;">Success
------ Write 7B to 10.XXX.XXX.XXX:39817-&gt;10.XXX.XXX.XXX:56789 (java)</span>

<span style="color: #0000ff;">Success
------ Write 7B to 10.XXX.XXX.XXX:42907-&gt;10.XXX.XXX.XXX:56789 (java)
</span></pre>

**WAIT - don’t read the next paragraph.** Let’s see how good your sleuthing skills are. The answer to our problem is in the screenshot above. Can you see what’s wrong with the HTTP response?

  
  
Here’s the problem: *text/plain: content-type* should read *content-type: text/plain*. It was backwards! :). This is an illegal HTTP response and HAPROXY was rejecting it.

### Conclusion

Troubleshooting can be hard. Monitoring tools are typically great for alerting you to a problem, and in many cases getting you very close to what might be the root cause. But many times, you have to get knee deep into it before you can really solve the problem. If you want to give your troubleshooting a leg up, add [open source sysdig][6] to your toolbox!

 [1]: http://rancher.com/
 [2]: http://www.sysdigrp2rs.wpengine.com
 [3]: http://www.haproxy.org/download/1.5/doc/configuration.txt
 [4]: https://sysdigrp2rs.wpengine.com/blog/troubleshooting-containers-when-theyre-gone/
 [5]: /wp-content/uploads/2016/08/tracecapture.png
 [6]: http://www.sysdig.org/