---
ID: 9746
post_title: >
  Monitoring and securing Java apps at
  Quby.
author: Eric Carter
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/monitoring-and-securing-java-apps-at-quby/
published: true
post_date: 2018-08-29 02:15:37
---
Moving to a Docker-based cloud for Java apps orchestrated by Mesos Marathon required a different approach to monitoring and security for <a href="https://www.quby.com/" target="_blank" rel="noopener">Quby</a>, the Amsterdam-based developer of smart home solutions and maker of smart thermostat and service platform 'Toon.' That's when they found Sysdig. **The <a href="https://sysdigrp2rs.wpengine.com/platform/" target="_blank" rel="noopener">Sysdig Cloud-Native Intelligence Platform</a> helps Quby resolve issues faster, and reduces monitoring system administration effort by 400%.**

### Business drives a move to the Cloud

In 2016, Quby began to experience significant growth as the popularity of its smart, in-home thermostats began to skyrocket. With growth came challenges. "Quby was a very classical ops kind of organization," explains Nicolas Kramer, Infrastructure Team Product Owner at Quby "It was a company that made everything themselves." When customer count rapidly expanded to 300,000, Quby’s datacenter infrastructure – designed to support 10,000 customers – was holding them back.

<p style="text-align: center;">
  <a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/QubyDisplay_Home_withbackground-72px.png"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/QubyDisplay_Home_withbackground-72px.png" alt="quby smart thermostat" width="568" height="368" class="wp-image-9854 aligncenter" /></a><br /><strong>Quby Toon smart energy platform</strong>
</p>

 

The pressure to keep pace with the demands of its business spawned a '<a href="https://www.quby.com/news/2017/8/9/the-three-principles-behind-qubys-cloud-strategy" target="_blank" rel="noopener">Move to the Cloud</a>' project. "The strategy was to take our existing applications into the cloud," describes Kramer. Quby achieved more scalability and stability. The company recognized that if they wanted to make the best use of cloud, they needed to move IT towards becoming a DevOps-based organization.

As a part of the shift to the <a href="https://aws.amazon.com/" target="_blank" rel="noopener">Amazon Web Services (AWS)</a> cloud, the company chose <a href="https://mesosphere.github.io/marathon/" target="_blank" rel="noopener">Mesos Marathon</a> for cloud orchestration, and <a href="https://www.docker.com/" target="_blank" rel="noopener">Docker</a> as the platform for applications. "We chose Mesos because it gave us a way to easily scale our services with demand. As we bring on more clients, the system is able to auto-adjust and make sure users have a good experience."

### Better monitoring = lower costs

After moving its applications and databases to AWS, Quby started looking at how to do monitoring in the cloud. "Moving to the cloud doesn't just mean, ‘Hey let's just run applications on someone else's computers,’ it also means changing your mindset on how you monitor applications, on how you optimize your environments, and on how you handle misbehavior of your applications, containers, and everything else."

The company started to use Sysdig Monitor as a way to monitor the health of the environment. "Primarily it was first just to see if things were going OK in the cloud. Little by little our use of Sysdig started growing from there," Kramer points out.

**The move to Sysdig massively reduced the effort and cost of managing and operating monitoring for Quby.** Prior to Sysdig, maintaining monitoring was a two-person job – *one to define, install and maintain all of the required agents and applications checks, and another to review that everything functioned properly*. According to Juan Morales, DevOps engineer at Quby, "It used to take two admins a lot of time along the whole year just to keep monitoring in place. To keep Sysdig updated takes less than 30 minutes each month.” [tweet_box design="default" float="none"]Sysdig Monitor and Sysdig Secure enable Quby to resolve issues faster, and reduce its #monitoring system administration 400% from a 2 FTE effort down to .5 FTE.[/tweet_box] 

"We didn't have time for games. We needed to deliver results quickly – last month, yesterday – so that was a really tipping point for Sysdig. With Sysdig, you only need to run a container on the host and it's done. We get monitoring and security at the container level, at the host level, and at the application level.”

**A huge part of the value of Sysdig solutions for Quby is that Sysdig Monitor and Sysdig Secure utilize a single point of instrumentation.** The company saves time and resources by delivering monitoring, security, troubleshooting, and forensics from the same software agent. "If you're moving into the cloud, there's so much about the process, the people, the organization you need to change that every tiny win in time and every tiny win in energy is highly appreciated. The way Sysdig delivers security means we have one thing less to worry about," concludes Kramer.

### Monitoring and securing dynamic services

"Old-fashioned monitoring requires declaration of resources – usually at both ends," explains Morales. "You have to configure not just checks on the client and the server. You need to configure the clients on the server and the servers on the client, which means you need to know where the server is when you provision the client, which is something completely against the design paradigm we have in this new platform."

For Quby, everything is now defined as infrastructure-as-code. To meet the needs of a dynamic environment, they needed a monitoring solution that could do automatic discovery. "I don’t want to tell my monitoring system what my infrastructure looks like."

### Monitoring Java Applications

"We have some legacy applications, but [Java][1] is basically what you'll find around here," indicates Morales. Applications in use at Quby include Spring Boot, Apache Tomcat, GlassFish, and Liferay. "And not to forget OpenVPN," adds Kramer. "Every device, every display at the customer is connected through a VPN tunnel to our service center. Sysdig is important in monitoring that all these things are working. It's a good way for us to measure if it's our problem and our back end has broken down or if AWS has some EC2 problems."

<p style="text-align: center;">
  <a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/monitor-dashboard-updated.jpg"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/monitor-dashboard-updated.jpg" alt="" width="1590" height="1000" class="aligncenter wp-image-9620 size-full" /></a><br /><strong>Monitoring Java apps with Sysdig</strong>
</p>

 

### Exploring with Sysdig

"Sysdig has a lot of strong advantages in that it is monitoring, but it is also very exploratory. It allowed us to get acquainted with the system. It helped in educating the team when we were getting into cloud orchestration," emphasizes Kramer. Sysdig Monitor allows Quby to bridge and empower different stakeholders and share information cross-company. It provides visibility for monitoring and operational teams in addition to development.

"With Sysdig we have a single place with a common language," notes Morales. "We are also on-boarding management teams. Let's say the CFO is having some questions about our infrastructure, we can bring up Sysdig and show our dashboards."

### Securing the Cloud with Sysdig Secure

Quby recently added Sysdig Secure to its environment to aid with container security monitoring and forensics. "Sysdig Secure was very well-timed for us in the sense that we were looking internally at what kind of things we needed to do with cloud security," reveals Kramer. One of the key security challenges identified by Quby was the complexity of performing investigation post incident. "

<p style="text-align: left;">
  In our search for a solution it was very hard to find something that took into consideration the aftermath of a problem," says Kramer. "It is complex to collect the logs and follow the tracks of what happened. You're really happy when you see that something went wrong because you can learn from it. In the cloud more often than not, things break and self-heal without you noticing much. You still want to see what happened. Sysdig Secure is a great fit to solve this from a runtime security and forensics standpoint."
</p>

<p style="text-align: center;">
  <a href="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/quby-secure-inspect.png"><img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/08/quby-secure-inspect.png" alt="quby sysdig secure inspect" width="1997" height="1141" class="alignnone size-full wp-image-9817" /></a><br /><strong><br />Sysdig security and forensics</strong>
</p>

 

### Winning customers with Sysdig

**Sysdig helps Quby provide confidence to clients who have questions about the service that Quby has built.** Kramer explains, "Let's say we have a potential customer coming along. The customer will often have concerns like, ‘You will be tendering to a million of my clients. You need to prove that your environments are stable and rock-solid.’ With Sysdig we can provide a level of transparency. We can show that we have ownership of the environment in a way that very few companies can."

"Our relationship with Sysdig feels very much like a good partnership. For us agility within the companies we rely on is very important. Help from Sysdig is always quite speedy. Sysdig as an organization has shown us a lot of flexibility and we have found big benefits in working with a relatively new product and company."  
  
[Download the full Quby case study here.][2]

 [1]: https://sysdigdocs.atlassian.net/wiki/spaces/Monitor/pages/204734661/
 [2]: https://go.sysdigrp2rs.wpengine.com/case-study-quby