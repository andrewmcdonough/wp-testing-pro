---
ID: 1570
post_title: >
  Sysdig Cloud for Cassandra, ActiveMQ,
  Tomcat, Custom JMX Metrics, and Much
  More
author: Marcus Sarmento
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/sysdig-cloud-for-cassandra-activemq-tomcat-custom-jmx-metrics-and-much-more/
published: true
post_date: 2015-05-07 06:00:17
---
A few months back, Sysdig Cloud <a href="https://sysdigrp2rs.wpengine.com/blog/gain-insights-jvms-sysdig-clouds-jmx-metrics/" target="_blank" rel="noopener">announced support for JVM metrics</a>, and the response has been overwhelmingly positive! The reason this is resonating with our customers is because they are now able to analyze rich infrastructure data and augment that with application-level metrics all automatically correlated in the same Sysdig Cloud views. Monitoring the performance of your systems will always be important but at the end of the day application performance is what really matters to your end users. Giving you full context across your infrastructure so you can quickly pinpoint performance bottlenecks, no matter where they are located, is a very important part of ensuring an optimal end user experience. 

I’m glad to say that our engineering team has been busy expanding our JMX capabilities and, starting today, you’ll notice substantial enhancements to our JMX support. This improved visibility into application performance metrics is the next step towards our goal to provide you with a world-class solution that offers a holistic view of your entire infrastructure. 

## What’s New? 

Before I walkthrough the new functionality it’s worth noting that all of these new metrics have the same one-second granularity, process-level data, and automatic correlation to application and system metrics from across your infrastructure that you’ve grown accustomed to. 

The first improvement is support for **new arbitrary JMX metrics**. Any and all custom JMX metrics you’ve already coded or will code into your application in the future can now be pulled into the Sysdig Cloud user interface. With a simple <a href="http://support.sysdigrp2rs.wpengine.com/hc/en-us/articles/204178959-JMX-Metrics-Configuration" target="_blank" rel="noopener">agent configuration file update</a>, you can pull in any metric from your Java applications and automatically correlate that with the rest of your infrastructure to surface trends and performance bottlenecks inside your Java environments. 

The next way we’ve enhanced our JMX support is by including **new default metrics** for commonly used Java-based technologies. If you’re using or plan on using <a href="http://cassandra.apache.org/" target="_blank" rel="noopener">Cassandra</a>, <a href="http://tomcat.apache.org/" target="_blank" rel="noopener">Tomcat</a>, <a href="http://activemq.apache.org/" target="_blank" rel="noopener">ActiveMQ</a>, <a href="https://zookeeper.apache.org/" target="_blank" rel="noopener">ZooKeeper</a>, <a href="http://kafka.apache.org/" target="_blank" rel="noopener">Kafka</a>, or <a href="http://hbase.apache.org/" target="_blank" rel="noopener">HBase</a>, we now import default metrics for each automatically. No additional plugins or configuration required! 

Finally, we’ve **added built-in views** for all of the new Java-related metrics we support out of the box. Quickly get a detailed view of your Java-based deployments with pre-built dashboards that display metrics for technologies like Cassandra, Tomcat, and ActiveMQ. Not only do we provide deep visibility into Java deployments, we make it extremely easy to setup powerful alerts directly from these built-in views (check out our <a href="https://sysdigrp2rs.wpengine.com/blog/alerting-todays-tomorrows-distributed-containerized-environments/" target="_blank" rel="noopener">alerting blog</a> for more information). 

## Conclusion 

With this new JMX functionality, you can understand how your Java-based applications interact with the rest of your infrastructure, with one-second granularity, so you can monitor and proactively manage performance in one, unified view. We’re very excited about our enhanced support for JMX metrics and we’re confident you’ll gain value from this functionality in your own environments. I encourage you to <a href="https://sysdigrp2rs.wpengine.com/landing-pitch2/?utm_source=web&utm_medium=blog&utm_campaign=jmxv2blogcta" target="_blank" rel="noopener">sign up for Sysdig Cloud</a> to test drive our new and improved JMX support for yourself!