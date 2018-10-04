---
ID: 7679
post_title: 3 phases of Prometheus adoption.
author: Loris Degioanni
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/3-phases-of-prometheus-adoption/
published: true
post_date: 2018-06-29 11:57:04
---
## How to ensure visibility into your next-generation Kubernetes environment.

<span>Having assisted hundreds of enterprises in developing a new visibility strategy as they move to Kubernetes, I’ve learned a few things about how organizations learn, evolve and adopt a new method of application observability. Open source is usually essential to developing this understanding</span>.

<span>In the cloud-native monitoring world, Prometheus is widely considered the place to start. Just like Kubernetes is the leading open source container orchestrator in the cloud-native world, Prometheus is the leading software choice for open source cloud-native monitoring. If you’re looking for more detail on what Prometheus is and how it works, read this</span> <a href="https://sysdigrp2rs.wpengine.com/blog/prometheus-monitoring-and-sysdig-monitor-a-technical-comparison/" target="_blank" rel="noopener">nice Prometheus monitoring summary</a>.<span>While most organizations end up not using unsupported open source in production, many start here. Where you end up depends on your own business’s requirements. Let me briefly cover the three phases I typically see enterprises go through on their way to a production-ready strategy.</span>

### 1\. Experiment {#1experiment}

<span>Much like the adoption of containers or Kubernetes doesn’t happen overnight, neither will an accompanying visibility strategy. The good news is that with Prometheus, your developers can explore without constraints of time or budget</span>.

At this phase you're looking for :

*   What does it take to get basic instrumentation in place?

*   Will it work for our own software?

*   What kind of alerts can I create?

*   How much detail does it provide for root cause analysis?

<span>Prometheus might require some setup in terms of its instrumentation model or its query language, but that’s a low cost to pay for freedom of exploration. Alternatively, commercial products may offer free tiers or trials with more constraints on time or features, but no management overhead.</span>

### 2\. Instrument {#2instrument}

<span>With some basic experimentation out of the way, it’s time to get more data into the system and understand what this looks like with a slightly broader deployment. To do that, consider two forms of instrumentation</span>:

*   <a href="https://prometheus.io/docs/instrumenting/exporters/" target="_blank" rel="noopener">Use exporters</a>. <span>The Prometheus community has dozens of exporters designed to simplify scraping metrics from common software components that already expose metrics through an endpoint. They can easily be deployed through Kubernetes in a systematic way. These exporters use a push model, which may be problematic security-wise depending on the complexity of your production environment</span>.

*   <a href="https://prometheus.io/docs/instrumenting/clientlibs/" target="_blank" rel="noopener">Use the Prometheus metrics format</a>. <span>Prometheus specifies a container and microservices friendly format that allows you to emit custom metrics directly from your applications. This is essentially to enable you to deeply observe your own code. Note that if your development team already uses a metrics format like StatsD or JMX, you may be able to use that already, but it will likely require greater operational effort and have reduced functionality. More on that in the next section</span>.

<span>OK, so now your team is gaining confidence and you’ve got the actual metrics you want, we’re done right? Not quite. Time to get into production</span>.

### 3\. Operationalize {#3operationalize}

<span>This might come as a shocker, but your experiments will likely look nothing like your own real-world production environment 12 months down the road. Let me run through a few of the critical questions for you to consider before you operationalize your new monitoring strategy so you’re not blindsided later</span>:

*   <span>Does the solution meet scaling requirements? How much data is coming in, and how long do you want to store it? Are you comfortable managing many databases or do you want it all centralized</span>?

*   <span>How will you control access to the data? Will you limit data by asking development teams to meet security and compliance requirements</span>?

*   <span>What happens with metrics that aren’t in Prometheus format? Will you find a way to support them, especially as legacy applications move to containers</span>?

*   <span>Will the query language model work as you grow? PromQL is powerful and flexible, and it will be easily adopted among your developers with time. But do your platform operations and support teams have those skills? Will you make the effort to teach them what they need to know</span>?

*   <span>And finally, how much resource are you willing to spend in ongoing maintenance? Every system requires some. You’ll need to decide if you want to pay that cost through human resources, through capital resources in the form of Prometheus support, or by licensing a commercial product</span>.

<span>Along the way, you invariably will have to answer the classic build-versus-buy question: Should you build something from open source and maintain it yourself, or buy something that will allow you to migrate your experiment easily and simplify your life going forward</span>?

<span>These questions will help define your monitoring strategy in a successful manner for your next generation container platform. With a carefully measured approach, you’ll be able to grow into a successful Prometheus deployment and also figure out the correct support model to build for your organization</span>.

*Originally featured on <a href="https://www.infoworld.com/article/3275887/containers/3-phases-of-prometheus-adoption.html" target="_blank" rel="noopener">InfoWorld, May 23, 2018</a>*