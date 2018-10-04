---
ID: 7231
post_title: 'Auditing container activity &#8211; A real example with wget and curl using Sysdig Secure.'
author: Knox Anderson
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/auditing-container-activity/
published: true
post_date: 2018-05-23 23:53:10
---
One of the first questions [Sysdig Secure][1] and [Sysdig Falco][2] users have is how do I audit container activity or detect when **X** happens inside a container, on a host, or anywhere in my environment. In today’s example we’ll cover detecting curl and the use of other programs that fetch contents from the web. There are illegitimate reasons to execute curl, such as downloading a reverse shell or a rootkit, but there are many legitimate reasons as well. In either case detecting the usage of web fetch programs in general is something many organizations need to do for compliance reasons.

If you’re new to the Falco rules syntax I recommend you check out some of these great resources to get up and running:

*   The Falco <a href="https://github.com/draios/falco/wiki/Falco-Rules" target="_blank" rel="noopener">wiki</a> - check out the default rules and learn the basics of lists, macros, etc

*   The Sysdig <a href="https://github.com/draios/sysdig/wiki/Sysdig-User-Guide" target="_blank" rel="noopener">wik</a><a href="https://github.com/draios/sysdig/wiki/Sysdig-User-Guide" target="_blank" rel="noopener">i</a> - learn the filtering syntax that falco rules use to detect unwanted behavior

*   Blog - <a href="https://sysdigrp2rs.wpengine.com/blog/getting-started-writing-falco-rules/" target="_blank" rel="noopener">Getting started writing falco rules</a>

*   Blog - <a href="https://sysdigrp2rs.wpengine.com/blog/friends-dont-let-friends-curl-bash/" target="_blank" rel="noopener">Friends don't let friends curl | bash</a>

*   A Katacoda <a href="https://katacoda.com/sysdig/scenarios/sysdig-falco" target="_blank" rel="noopener">scenario</a> that goes through the steps of creating and triggering falco rules.

## Name the Web Fetch Programs

First, create a list + macro that names the programs that count as "web fetch programs". Here's one possibility. By convention, "binaries" are used to name programs, and "programs" refer to a condition that compares proc.name to a list of binaries. Also note the good practice of parentheses surrounding the condition field of each macro to ensure it is always treated as a single unit.

<pre>list: web_fetch_binaries
items: [curl, wget]</pre>

<pre>macro: web_fetch_programs
condition: (proc.name in (web<em>fetch</em>binaries))</pre>

## Capture Spawning a Web Fetch Program {#capturespawningawebfetchprogram}

Next, write a macro that captures any exec of a web fetch program. Here's one way, using the existing spawned_process macro:

<pre>macro: spawned_process
condition: evt.type = execve and evt.dir=&lt;
</pre>

*If you have any questions about the filtering syntax on the condition check out some of the [available filters documented here][3]*

Next combine the web_fetch_programs and spawned_process macros, to have a single macro that defines "curl or wget were executed on my system":

<pre>macro: spawn_web_fetcher
condition: (spawned_process and web_fetch_programs)
</pre>

## Add exclusions, define the rule {#addexclusionsdefinetherule}

When creating this rule we’ll want to add a macro that adds the ability to name a set of exceptions and combine that with the macros we created earlier.

Note use of the container macro to restricts the rule to containers. To monitor for this behavior on all hosts and inside containers just remove the container macro in the condition below.

<pre>macro: allowed_web_fetch_containers
condition: (container.image startswith quay.io/my-company/services)
</pre>

rule: Run Web Fetch Program in Container desc: Detect any attempt to spawn a web fetch program in a container condition: spawn_web_fetcher and container and not allowed_web_fetch_containers output: Web Fetch Program run in container (user=%user.name command=%proc.cmdline %container.info image=%container.image) priority: INFO tags: [container]

Now you’re all set to use this rule in your environment with falco. Next we’ll cover adding this rule to Sysdig Secure so you can have more controls over the scope of where a policy applies, take actions like killing or pausing a container, or record all the system activity before and after this web fetch event for forensic analysis.

To add this rule to Sysdig Secure copy the rule and exclusion into the custom rules section, and then save it be able to add the rule `Run Web Fetch Program in Container` to a policy.

![Detecting Curl][4]

## Create Policy Associated With Rule {#createpolicyassociatedwithrule}

Now, create a Sysdig Secure policy associated with the rule.

![Detecting Curl][5]

*This policy can be scoped by container and orchestrator labels so you can apply it to different areas of your infrastructure depending on their security and compliance needs. Actions like killing a container can also be taken.*

By switching to the falco tab within the policy you can select the new rule `Run Web Fetch Program in Container` to apply it to the policy you just created. 

![Detecting Curl][6]



## Verify Policy Is Working {#verifypolicyisworking}



Finally, a command like docker run --rm byrnedo/alpine-curl <a href="https://www.google.com/" target="_blank" rel="noopener">https://www.google.com</a> should result in a policy event:



![Detecting Curl][7]



*Within this policy event you’ll get full context of what triggered the event and where it occurred in your physical and logical infrastructure.*



Hopefully this example provides an easy way to get started writing your first falco rules to audit the activity occurring inside your containers. If you need help contact us on [Slack][8] and share any rules you create with the community by submitting a PR to the [Sysdig Falco][9] project.

 [1]: http://sysdigrp2rs.wpengine.com/products/secure
 [2]: https://sysdigrp2rs.wpengine.com/opensource/falco/
 [3]: https://github.com/draios/sysdig/wiki/Sysdig-User-Guide#user-content-filtering
 [4]: /wp-content/uploads/2018/05/Detecting_curl_1.png
 [5]: /wp-content/uploads/2018/05/Detecting_curl_2.png
 [6]: /wp-content/uploads/2018/05/Detecting_curl_3.png
 [7]: /wp-content/uploads/2018/05/Detecting_curl_4.png
 [8]: http://sysdig.slack.com
 [9]: https://github.com/draios/falco/pulls