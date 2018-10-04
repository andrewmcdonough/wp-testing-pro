---
ID: 10334
post_title: 'How to instrument code: Custom metrics vs APM vs OpenTracing.'
author: Jorge Salamero Sanz
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/how-to-instrument-code-custom-metrics-vs-apm-vs-opentracing/
published: true
post_date: 2018-09-19 01:20:52
---
Custom Metrics (JMX, Golang expvar, Prometheus, statsd or many other), APM and Opentracing are different approaches on how to instrument code in order to monitor health, performance and troubleshoot your application more easily.

To instrument your code in your application, you need to understand the differences between the different options and their pros and cons in each use case; in this guide we are going to cover how to create custom metrics in Java, Golang, Javascript and Python, so you can make yourself an idea about how it works.

## Why do we need code instrumentation? {#whydoweneedcodeinstrumentation}

We all agree that creating an application and not monitoring it, is not a good idea. I’ve personally seen projects where they weren't monitoring the application; everything crashed, they had no idea about what was going on and troubleshooting was chaos. No one knew what was going on and it was crashing on a per day basis in production, creating a terrible user experience and calling the sysadmin team at 3 AM complaining that they couldn’t work. It was a mess. If a monitoring and alerting process was in place, troubleshooting would have been easy.

For example, if you are developing an online shop you are going to gain insights on how your application performs in production: which are your most visited products, the slowest to load products, which users are experiencing slowness, what device, browser or geography may be slow, how long does it take to load the frontend part versus the backend database, to mention a few typical use cases. To gain this visibility you can either use an APM tool, use OpenTracing libraries or generate metrics ad-hoc for these parts you are interested in.

In this blog post series we are going to compare all the custom metric instrumentation options (Prometheus, expvar, statsd and JMX) showing examples on how to implement them in different programming languages: Golang, Java, Python, and Javascript. We will also compare using custom metrics with using APM and OpenTracing, so you can understand what do you exactly need.

## Comparing Custom Metrics, APM and OpenTracing: How to instrument code? {#comparingcustommetricsapmandopentracingwhatcodeinstrumentationshouldiuse}

Are custom metrics the same as APM? No, both are often complementary. Organizations sometimes need both to monitor and troubleshoot large-scale, distributed or complex applications, but many times they can do their job with infrastructure monitoring plus custom metrics.

It’s always important that application developers understand the behaviour of their application and how it performs so they can find issues and solve them quickly. They can do that leveraging an APM that gives you an overview of transactions or instrumenting their code through custom metrics to gain observability on the parts they are keen on inside their code.

APM tools are useful for 'detecting and isolating' a problem and enabling a code developer to troubleshooting at a code level. However, as we know, most production performance issues are not code-related but infrastructure and application related. APM tools seldom go deep to the layers below the application to help determine the root cause. APM can tell the what is slow , you need more full stack telemetry to determine why its slow.

There are multiple reasons why your application can fail and are completely unrelated to code and can’t be covered just using APM even if includes lightweight infrastructure monitoring, for example:

*   Your deployments that have JVMs have high heap usage.
*   Your disk available to an specific application is at its full capacity.
*   Some nodes in your cluster are oversubscribed.
*   Kubernetes is not running all the replicas you requested for a given deployment.
*   A daemon that your application depends on is failing.
*   Some other application is using too much CPU
*   You are under a DDoS attack.
*   Kafka queues are backed up.
*   Stolen CPU is high on adjacent VMs in the cluster.
*   Certain processes are over utilizing network bandwidth within the same namespace.
*   Cassandra compactions have dropped indicating not enough data being backed up for your services.
*   Network bottleneck caused by dropped packets.

While APM can help in a situation like:

*   A page takes longer than expected to load, but only sometimes
*   You need to identify which parts generate slow queries on your backend

A right DevOps strategy requires full stack visibility across all stages of the application lifecycle, but APM itself does not provide that; APM data just represents a small subset of all the information that a infrastructure monitoring system can process.

So, when do you need APM and when you should implement custom metrics instead? It depends.

If you are early in your development process and you know which parts can be problematic you can start implementing custom metrics from the beginning. If you know or you can guess where you problem comes from you can instrument only that part of your code base.

If you need metrics all over the code and per transaction observability APM is what it takes then. But if you don’t need per transaction because you are doing server-side microservices monitoring, a full stack scalable telemetry tool that encompasses APM golden signals, application metrics and low level infrastructure and network metrics would be more valuable.

APM at the same time is more resource intensive, sometimes actually can become quite hungry, so that needs to be considered when running in production environments. That’s why you will find that often only a subset of request are sampled. Custom metrics instead, are definitely more lightweight.

OpenTracing is an standard for distributed tracing. We can see this as the instrumentation part of an APM. Ideally you instrument your code one time and then you should be able to visualize your data in multiple places, including open-source UIs or commercial APMs.

## Comparison table between Custom Metrics and APM {#comparisontablebetweencustommetricsandapm}

All this sounds like too many if… can you give me a table so we can get this quickly? You got it.

<table>
  <thead>
    <tr>
      <th>
      </th>
      
      <th>
        Custom Metrics
      </th>
      
      <th>
        APM
      </th>
      
      <th>
        OpenTracing
      </th>
    </tr>
  </thead>
  
  <tbody>
    <tr>
      <td>
        Code-related problems
      </td>
      
      <td>
        Devs need to provide metrics with performance in code but are not as easy to identify
      </td>
      
      <td>
        Yes
      </td>
      
      <td>
        Yes
      </td>
    </tr>
    
    <tr>
      <td>
        Infrastructure-related problems
      </td>
      
      <td>
        Yes
      </td>
      
      <td>
        No
      </td>
      
      <td>
        No
      </td>
    </tr>
    
    <tr>
      <td>
        Node and service level aggregation
      </td>
      
      <td>
        Yes
      </td>
      
      <td>
        No
      </td>
      
      <td>
        No
      </td>
    </tr>
    
    <tr>
      <td>
        Standard implementaton
      </td>
      
      <td>
        Some languages include a standard way to implement them: (Prometheus, Java JMX, Go expvar, ...)
      </td>
      
      <td>
        No
      </td>
      
      <td>
        Yes
      </td>
    </tr>
    
    <tr>
      <td>
        Allows capacity planning
      </td>
      
      <td>
        Yes
      </td>
      
      <td>
        No
      </td>
      
      <td>
        No
      </td>
    </tr>
    
    <tr>
      <td>
        Allows complete statistical measurements
      </td>
      
      <td>
        Yes
      </td>
      
      <td>
        No
      </td>
      
      <td>
        No
      </td>
    </tr>
    
    <tr>
      <td>
        Cloud Native Computing Foundation standard
      </td>
      
      <td>
        Prometheus metrics only
      </td>
      
      <td>
        No
      </td>
      
      <td>
        Yes
      </td>
    </tr>
    
    <tr>
      <td>
        Distributed application analysis
      </td>
      
      <td>
        Yes, without per trace analysis
      </td>
      
      <td>
        Yes
      </td>
      
      <td>
        Yes
      </td>
    </tr>
    
    <tr>
      <td>
        Useful for developers for pre-production environments
      </td>
      
      <td>
        Yes
      </td>
      
      <td>
        Yes
      </td>
      
      <td>
        Yes
      </td>
    </tr>
    
    <tr>
      <td>
        Useful for complete DevOps strategy
      </td>
      
      <td>
        Yes
      </td>
      
      <td>
        No
      </td>
      
      <td>
        No
      </td>
    </tr>
  </tbody>
</table>

##   
Custom Metrics instrumentation examples {#customcodemetricinstrumentationexamples}

There are multiple ways to instrument your code with custom metrics. Some languages like Java and Golang provide standard ways to do so, native to those languages, while for others like Javascript or Python, you will require third-party libraries (e.g. Prometheus or Statsd) to feed your monitoring system.

We’ve prepared some custom metric instrumentation examples for you about how to implement them in:

*   How to instrument Java code (with JMX custom metrics)
*   How to instrument Go code with custom expvar metrics
*   How to instrument statsd metrics in Go, Java, Python or Javascript (with code examples)
*   Prometheus with Go, Java, Python and Javascript with examples

## Conclusions {#conclusions}

As you can see there are multiple different ways to instrument your application. Each one has it’s benefits and it’s drawbacks and they are all focused in different approaches.

APM gives you transaction level observability and focuses on the code, custom metrics requires to know what you are doing, because it's a more DIY but won’t give you per transaction observability, instead will provide full stack visibility: hosts, orchestration, services and applications.

You can use one, some or all of them to throw all the light to your application and be confident that’s performing correctly, will depend on your needs.