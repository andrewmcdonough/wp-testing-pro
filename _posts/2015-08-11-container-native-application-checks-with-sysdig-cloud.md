---
ID: 1969
post_title: >
  Container-Native Application Checks with
  Sysdig Cloud
author: Marcus Sarmento
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/container-native-application-checks-with-sysdig-cloud/
published: true
post_date: 2015-08-11 07:00:33
---
## Introduction

At Sysdig we take great pride in providing an exceptional experience right out of the box for customers leveraging container technologies. Our intuitive UI and minimal configuration requirements mean that you get value in minutes instead of hours or days. But we’re not stopping there! Sysdig Cloud has enhanced our native support for containerized environments by adding application checks for many of today’s most commonly used IT technologies: ElasticSearch, HAProxy, Redis, Memcached, RabbitMQ, and dozens of others. And with our unique ContainerVision™ technology, Sysdig Cloud enables you to collect rich metrics from your containerized applications with no additional instrumentation and nothing to pollute your containers. 

The great part about this enhanced support is that Sysdig Cloud now **automatically** detects that you are running these applications and **automatically** starts collecting, processing, and analyzing these metrics without you having to do any additional configuration or install any additional plugins. And these aren’t just shallow metrics we are collecting, we’re providing you all of the info you need to properly manage the performance of these different applications with full context of all the different interactions across your containerized environment. 

Some example application technologies we detect and metrics we collect include: 
## Elasticsearch

*   elasticsearch.cluster_status
*   elasticsearch.http.current_open
*   elasticsearch.indexing.index.time
*   elasticsearch.number_of_data_nodes
*   elasticsearch.thread_pool.snapshot.queue
*   plus more, **over 90 Elasticsearch metrics total!**

## HAProxy

*   haproxy.backend.response.time
*   haproxy.backend.connect.time
*   haproxy.backend.queue.current
*   haproxy.frontend.errors.req_rate
*   haproxy.frontend.bytes.in_rate
*   plus more, **nearly 40 HAProxy metrics total!**

## Redis

*   redis.aof.last_rewrite_time
*   redis.clients.biggest_input_buf
*   redis.info.latency_ms
*   redis.mem.peak
*   redis.slowlog.micros.95percentile
*   plus more, **over 35 Redis metrics total!**

## Memcached

*   memcache.bytes_written_rate
*   memcache.cmd_flush_rate
*   memcache.curr_connections
*   memcache.fill_percent
*   memcache.uptime
*   plus more, **over 25 Memcached metrics total!**

## RabbitMQ

*   rabbitmq.node.fd_used
*   rabbitmq.node.mem_used
*   rabbitmq.queue.messages.publish.rate
*   rabbitmq.queue.messages_ready
*   rabbitmq.queue.messages_unacknowledged
*   plus more, **over 15 RabbitMQ metrics total!**

## Container-Native Metric Collection

Where this enhanced application support starts to get really interesting is inside containerized environments. Traditionally, getting access inside containers to be able to see what applications are running require you to install agents on each container. This isn’t ideal for a number of reasons; security risks, the inability to leverage images from Docker Hub, and at the end of the day this goes against the Docker philosophy of running only one process inside each container. Sysdig Cloud allows you to get deep visibility inside containers without the need to instrument the container itself. Simply drop a Sysdig Cloud container on your host, and you’ll automatically see inside the rest of the containers and the infrastructure without any additional configuration. Think of it as <a href="https://sysdigrp2rs.wpengine.com/monitoring-as-a-microservice/" target="_blank">Monitoring as a Microservice</a>. 

By leveraging this patent-pending ContainerVision™ technology, we are able to see what application is running inside the container, without you having to add agents to each container. So if you are running Elasticsearch inside a Docker container for example, we’ll automatically detect Elasticsearch running and **start collecting metrics from inside the container, from outside the container itself** without you having to lift a finger! 

## Conclusion

Because our container-native technology does all of the heavy lifting, you are now able to automatically pull a comprehensive set of application metrics from inside containers, all without having to instrument or pollute your containers in any way. To try this new functionality out for yourself, sign up for a <a href="https://sysdigrp2rs.wpengine.com/landing-page/" target="_blank">free 14-day trial of Sysdig Cloud</a> today and let us know what you think!