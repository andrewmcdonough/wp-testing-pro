---
ID: 3317
post_title: 'Introducing Sysdig Teams: Service-based access control for simpler and more secure Kubernetes &#038; Docker monitoring'
author: Apurva Dave
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/introducing-sysdig-teams/
published: true
post_date: 2016-11-08 07:00:12
---
Like everyone these days, DevOps teams and developers are deluged with data. 

Making sense of it all is increasingly hard, even with the pretty analytics tools we now have available. Some of the data is irrelevant to your task, some of it is data you’re not supposed to see, and buried somewhere in there is the little nugget you care about. 

The move to microservice and containerized environments makes the data deluge worse. Containers create more data, and many microservices can leave all that data even more jumbled up than before. 

It’s painful and unproductive for you. And this additional data exposure could also create more risk for your company. 

We aimed to solve this problem in a way that fits naturally with the architecture as well as the workflows of modern DevOps environments. The result is Sysdig Teams! 

### Service-based Access Control



While you’re likely familiar with role-based access control, Sysdig teams introduce the concept of **service-based access control**. With service-based access control, administrators can define groups of users that have access to the dashboards, alerts, and data limited to a service or set of services, as defined by your orchestration system (think Kubernetes, Mesos, and the like). 

You might be asking, “Why is this important?” And you should be. Our answer is: Because this is the way your organization wants to work.  Service-based access control is ideal to reduce the exposure of data to those who actually need it, and also makes users more productive through focusing them on data that is relevant to them. 

Think about it: your dev teams form around an application or a service, and they frequently are thinking of your entire system in relation to their own focus. They want their alerts going to their own private Slack channel or PagerDuty channel, and others don’t necessarily want or need to see that. They want dashboards shared only among their team, isolating the key metrics that matter to them. And that particular dev team might configure their alerts to use different anomaly detection algorithms or logic than other teams. (That last point was a subtle plug for our anomaly detection capabilities, by the way.) 

Your platform team is thinking about how to support these development teams, and enable self-service for them, while at the same time keeping an eye on the entire operation. Your SecOps team is looking to isolate access to data while not impeding developer productivity. That’s a lot of demands, all at once, and service-based access control can address them. 

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/11/Teams-Blog_Illustration-8.jpg"><img src="/wp-content/uploads/2016/11/Teams-Blog_Illustration-8.jpg" alt="Sysdig Teams" width="750" height="536" class="aligncenter size-large wp-image-3299" /></a>



The alternative to service-based access control is setting up a separate, isolated docker container monitoring per dev team. Going down that route means more management work for everyone, less consistency, and also more things to secure. That just doesn’t seem like a viable solution at any reasonable scale.   

All in all, we think that the concept of services is the standard model of today’s distributed applications and infrastructure, and that means your container monitoring system should be thinking that way too. 

### Introducing Sysdig teams



With this new functionality in Sysdig Cloud, you’ll be able to **organize your users into teams**, which will enforce your data access security policies while improving your users’ troubleshooting workflows. Teams are **isolated from each other**, limit the exposure of dashboards and alerts, and control access to infrastructure, service, and application performance data. They simplify the process of a user getting from system login to the data they need right now. 

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/11/Teams-Blog_Illustration-10.jpg"><img src="/wp-content/uploads/2016/11/Teams-Blog_Illustration-10.jpg" alt="Sysdig Teams" width="750" height="536" class="aligncenter size-large wp-image-3299" /></a>



Sysdig creates teams by **dynamically filtering your metrics based on metadata** already present in your system. Sysdig already retrieves metadata from your orchestration system to aggregate docker container monitoring data into views for your deployments, services, and tasks. Now we also use that metadata to shape teams’ access to the underlying data. This provides your organization with an always-up-to-date method of isolating data that continually adapts to your changing infrastructure. 

Before we describe Teams in more detail, let’s take a look what our beta users are doing with this functionality. 

### Monitoring Scenarios for Teams: Microservices, Kubernetes, Security, and more



We’ve been happy to see a large number of potential uses for teams appear as customers tested this new functionality with us. Here’s a quick review of what we’ve heard so far. I’m sure some new ones appear as you get your hands on Teams! 

*   **The classic “dev vs prod” split:** Many organizations prefer to limit the number of people accessing data related to their production services. This is about isolating physical infrastructure and the applications on top.
*   **Microservices** where each team needs to only see their own dashboards and field its own alerts: By scoping down the data accessible to individual development teams and their related services, developers can more effectively focus on the information  that is relevant to them. This is about building teams based on logical isolation using orchestration or config management metadata.
*   **Platform as a service** where ops teams need to see the entire platform: This is somewhat the flip of the previous use case, enabling certain people to see all data for all services as well as the underlying hardware. This is perfect for managed service providers who are managing a multi-tenant environments, or devops teams using a similar model within their own organization.
*   **Restricted environments:** An even more specific use case of the microservices example to limit data access for security and compliance: Certain services, such as authentication and billing, may have a very specific set of individuals authorized to access them.
*   **Organizations or environments** that need to segment monitoring for efficiency: This is a wide ranging use case. We’ve seen very large organizations form teams just to simplify access; smaller organizations create ephemeral teams to troubleshoot a particular issue; or teams formed to optimize QA & support access to system data.



### How to set up Sysdig Teams



Teams are simple to set up, intuitive, and may be accessed via the UI or API. To set up a team, you really need to just do three things:



1.  **Set the scope** for a team based on orchestration metadata from Kubernetes, Mesos, and Swarm, as well as other characteristics of your environment.
2.  **Assign users** to any number of teams with fine-grained controls
3.  **Enable “Super-user” teams** that are perfect for devops admins or platform-as-a-service operators



[caption id="attachment_3558" align="aligncenter" width="721"]

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2016/11/Screen-Shot-2017-01-04-at-8.26.16-PM.png" alt="Sysdig Teams for service-based access control" width="721" height="467" class="wp-image-3558 " />][1] *Create service-based access via your monitoring metadata*[/caption]<p style="text-align: center;">
  <em> </em>
</p>

...and that’s it. You now have teams up and running. As a bonus, you can choose to leverage our new LDAP integration for more traditional access control and streamlined onboarding of users. LDAP is of course optional, but the integration is a nice feature if you already have it in place.



### Pricing & Availability



Teams are so valuable, that we think everyone should have them. As a result, all tiers of Sysdig Cloud will come with by default with two teams. There is no change to our base pricing.



Enterprise plans can customize the number of teams that they’d like.



Sysdig Teams are currently in limited release, and will be generally available later this quarter. If you’re a Sysdig customer and you’d like to test teams, just drop us a line and we’d be happy to add you to the beta program!

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2016/11/Screen-Shot-2017-01-04-at-8.26.16-PM.png