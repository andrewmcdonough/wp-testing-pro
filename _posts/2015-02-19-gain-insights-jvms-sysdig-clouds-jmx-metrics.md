---
ID: 10350
post_title: >
  Gain Insights into JVMs with Sysdig
  Cloud’s JMX Metrics
author: Marcus Sarmento
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/gain-insights-jvms-sysdig-clouds-jmx-metrics/
published: true
post_date: 2015-02-19 13:59:42
---
I’ve heard many users ask for it, and our amazing engineering team has delivered once again. With the latest release of Sysdig Cloud users have the ability to view and analyze **Java Management Extensions (JMX)** metrics out of the box with no additional configuration or setup required. By exposing these metrics from within **Java Virtual Machines (JVMs)**, customers that have Java components in their environment gain increased visibility and coverage of their heterogeneous application infrastructures.  
  
Exposing JMX metrics to users is helpful, but what makes Sysdig Cloud's approach unique is that we are able to leverage the same differentiated aspects of our solution and apply it to JMX metrics in a way that delivers huge value to operations teams responsible for managing Java application components.    
  
Some examples include:  
  
**Process-level metrics  
**Show JMX metrics and group them by process to add an additional layer of granularity that operations can utilize when troubleshooting Java performance.  
  
**One-second data streaming  
**Be the first to know when your JVM performance is abnormal with up to the second views.  
  
**Historical replay  
**View JMX metrics at any point in the past to retroactively troubleshoot performance degradations with DVR-like replay capabilities.  
  
**Holistic view of system, network, infrastructure, and application metrics  
**Automatically correlate JMX metrics with metrics from other parts of your distributed environment to understand the causes and effects of different incidents.  
  
By surfacing JMX metrics, Sysdig Cloud also facilitates more collaboration between operations teams and their development counterparts.  Imagine being able to tie an incident in production back to a particular garbage collection event.  Ops teams are happy because they can quickly isolate the root cause and get the ball rolling on a fix.  Dev teams are happy because they can finally pin point where they need to tune garbage collection performance as opposed to spending hours trying to re-create the issue on their own.  Everybody wins!  
  
The types of JMX metrics Sysdig Cloud exposes fall into four categories - Garbage Collection, Memory, Threading, and Loaded Classes.  Here's a comprehensive list of JMX metrics that Sysdig Cloud will now collect and display:  
### Garbage Collection

*   jvm.gc.cms.collectionCount
*   jvm.gc.cms.collectionTime
*   jvm.gc.cms.collectionCount
*   jvm.gc.cms.collectionTime
*   jvm.gc.parNew.collectionCount
*   jvm.gc.parNew.collectionTime
*   jvm.gc.psMarkSweep.collectionCount
*   jvm.gc.psMarkSweep.collectionTime
*   jvm.gc.psScavenge.collectionCount
*   jvm.gc.psScavenge.collectionTime

### Memory

*   jvm.heap.size
*   jvm.heap.used
*   jvm.nonHeap.size
*   jvm.nonHeap.used

### Threading

*   jvm.daemon.count
*   jvm.thread.count

### Loaded Classes

*   jvm.class.loaded.count

  
Users can visualize any of these metrics as a time series:  
  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/jmx-time-series2-1170x243.png" alt="" width="640" height="133" class="alignnone wp-image-10352 size-large" />  
  
A top 10 process table:  
  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/jmx-top-10-1170x245.png" alt="" width="640" height="134" class="alignnone wp-image-10353 size-large" />  
  
Or a map view:  
  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/jmx-map-view-1170x247.png" alt="" width="640" height="135" class="alignnone wp-image-10351 size-large" />  
  
We’ve also introduced a built-in view called “JVM Overview” which displays views for hosts and groups of hosts like heap usage over time, heap usage by process, garbage collection time, garbage collection count, and thread count:  
  
<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/09/jvm-overview-pic-1170x760.png" alt="" width="640" height="416" class="alignnone wp-image-10354 size-large" />  
  
If you haven’t already done so, feel free to <a href="https://sysdigcombak.wpengine.com/sign-up/?utm_source=web&utm_medium=blog&utm_campaign=jmxblog021915" target="_blank" rel="noopener">sign up for a free trial of Sysdig Cloud</a> to test drive these JMX metrics for yourself. Enjoy!