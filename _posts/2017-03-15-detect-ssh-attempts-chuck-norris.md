---
ID: 3842
post_title: >
  How to detect SSH attempts by Chuck
  Norris
author: Knox Anderson
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/detect-ssh-attempts-chuck-norris/
published: true
post_date: 2017-03-15 00:19:27
---
<script src="https://code.jquery.com/jquery-1.11.3.min.js"></script> <script src="https://code.jquery.com/mobile/1.4.5/jquery.mobile-1.4.5.min.js"></script> 
It’s fun to read about new tools on HackerNews, but I’ve always enjoyed getting my hands dirty and trying something new myself. That’s why we’re kicking off this series to give users an easy to follow, hands on experience using Sysdig to troubleshoot real problems.

Let’s get started with a scenario where we need to to find out the IP address and username of someone trying to SSH into our system. **We want you to find your own path to the bug so we won’t tell you what to do right away, but we will offer some clues to get you on track!** Each exercise takes a common troubleshooting use case and packages it up in a Sysdig capture, which is a file of all the system calls that happened on a host at a given point in time.

### Before we begin

#### Setup

1.  Install <a href="http://www.sysdig.org/install/" target="_blank">Sysdig troubleshooting tool</a>
2.  [Download Trace Capture 1][1]
3.  Check out this <a href="https://sysdigrp2rs.wpengine.com/blog/linux-troubleshooting-cheatsheet/" target="_blank">Linux Troubleshooting Cheatsheet</a> (if you’ve used `strace`, `tcpdump`, `htop`, `lsof`, `iftop`, or any similar tools. This will be gold to you ☺!

#### A little bit about Sysdig

*   <a href="https://sysdigrp2rs.wpengine.com/blog/sysdig-vs-dtrace-vs-strace-a-technical-discussion/" target="_blank">Sysdig vs DTrace vs Strace: a Technical Discussion</a>

#### Find the Imposter

*   **Environment:** A Kubernetes cluster running tons of stuff: Wordpress, Tomcat, MySQL, Cassandra, Redis, MongoDB... with each service replicated via Deployments, and exposed via Services
*   **Problem:** Someone repeatedly tried to log into the host via SSH and kept failing
*   **Goal:** Find the IP address and username of the imposter

#### Hints

*   **Tip:** To solve the problem, focus on what kind of I/O activity the SSH daemon does when serving a login request
*   **Tip:** Relevent Sysdig chisels: `topconns` and `echo_fds`
*   **Tip:** Relevant Sysdig filters: ` fd.port`, `fd.directory`, `fd.name`

I always like to start off by opening our capture in `csysdig` to just get an idea of what we’re looking at. To do that enter:

    $ csysdig –r chuck_norris_capture.scap
    

<a data-rel="lightbox-0" href="/wp-content/uploads/2017/03/Screen-Shot-2017-03-14-at-11.07.04-PM.png"><img src="/wp-content/uploads/2017/03/Screen-Shot-2017-03-14-at-11.07.04-PM.png" alt="csysdig" width="750" height="536" class="aligncenter size-large wp-image-3299" /></a>

As you can see here we have thousands of processes running on this machine. Let’s next check out what’s containers are running here by applying the container `csysdig` view: `[Fn + F2]`

<a data-rel="lightbox-0" href="/wp-content/uploads/2017/03/Screen-Shot-2017-03-14-at-11.08.50-PM.png"><img src="/wp-content/uploads/2017/03/Screen-Shot-2017-03-14-at-11.08.50-PM.png" alt="csysdig container view" width="750" height="536" class="aligncenter size-large wp-image-3299" /></a>

This host is jam packed with containers as well. Looking at the bottom right corner shows that there are 32 different containers running on this host. Since there’s so much data if we want to get an answer here we’ll need to get smart with the different filters and chisels.

<div>
  <h4>
    Hint #1
  </h4>
  
  <button type="button" class="btn btn-info" data-toggle="collapse" data-target="#hint1">Click for commands to inspect network activity</button> <div id="hint1" class="collapse">
    <p>
      Since we know that SSH is doing some sort of network activity that’s a good place to start.
    </p>
    
    <p>
      The first possible solution here is to check the variety of network connections we have on port 22 (the port on which SSH access occurs)
    </p>
    
    <pre><code>$ sysdig –r chuck_norris_capture.scap –c topconns fd.port=22</code></pre>
    
    <p>
      This command will read the capture file, apply a top connections chisel, and apply the filter file descriptor equals port 22.
    </p>
    
    <pre><code>
Bytes               Proto               Conn                
--------------------------------------------------------------------------------
4.53KB              tcp                 73.2.104.195:29842->10.1.1.214:ssh
4.46KB              tcp                 73.2.104.195:35925->10.1.1.214:ssh
4.33KB              tcp                 73.2.104.195:55820->10.1.1.214:ssh
856B                tcp                 73.2.104.195:57352->10.1.1.214:ssh
200B                tcp                 73.2.104.195:33762->10.1.1.214:ssh
80B                 tcp                 73.2.104.195:34435->10.1.1.214:ssh
</code></pre>
    
    <p>
      We can see that there are a bunch of connections on port 22. But they all seem to come from the same host: <code>73.2.104.195</code>. So probably it’s a fair guess that the IP address of the bad person trying to break into our system is this one.
    </p>
    
    <p>
      <code style="display: none;"> </code>
    </p>
  </div>
</div>

<div>
  <h4>
    Hint #2
  </h4>
  
  <button type="button" class="btn btn-info" data-toggle="collapse" data-target="#hint2">Click for commands to inspect network activity</button> <div id="hint2" class="collapse">
    <p>
      Once we have found the network connections, what else can we do? What happens if we inspect these connections? We know that there’s traffic on port 22. Now let’s see the the content.
    </p>
    
    <p>
      <i>(Pro tip: this can be done with sysdig or csysdig, but the since the file is so large lets use sysdig)</i>
    </p>
    
    <pre><code>sysdig -r chuck_norris_capture.scap -A -c echo_fds fd.port=22</code></pre>
    
    <p>
      <a data-rel="lightbox-0" href="/wp-content/uploads/2017/03/ssh-chisel.gif"><img src="/wp-content/uploads/2017/03/ssh-chisel.gif" alt="echo fds" width="750" height="536" class="aligncenter size-large wp-image-3299" --="" /></a>
    </p>
    
    <p>
      We can see all the interactions between the host and our local VM SSH. But, of course, SSH is supposed to be a secure protocol. By the time the data hits the kernel and the network, it has already been encrypted. The only plaintext thing we can see is this useless string of algorithm that SSH is exchanging. So looking at the network activity doesn’t seem to bring us a lot of value.
    </p>
  </div>
</div>



<div>
  <h4>
    Hint #3
  </h4>
  
  <button type="button" class="btn btn-info" data-toggle="collapse" data-target="#hint3">Bonus tip: think of the command we just used, but filter for activity in var/log</button> <div id="hint3" class="collapse">
    <p>
      Since we’ve crossed network activity off the list, let’s think what other activities SSH does that would generate a lot of activity. What about log file activity?
    </p>
    
    <pre><code>sysdig -r chuck_norris_capture.scap -A -c echo_fds fd.name contains auth.log</code></pre>
    
    <p>
      <a data-rel="lightbox-0" href="/wp-content/uploads/2017/03/chuck-norris.gif"><img src="/wp-content/uploads/2017/03/chuck-norris.gif" alt="ssh activity" width="750" height="536" class="aligncenter size-large wp-image-3299" /></a>
    </p>
    
    <p>
      This brings us much more useful information. We can see that <code>rs</code>, which is the <code>rsyslog</code> process, wrote a bunch of things in the <code>auth.log</code> file, among which are: <code>Failed password for invalid user chucknorris</code> and the IP address that we had found earlier.
    </p>
  </div>
</div>



#### Solution: Detecting SSH User Activity Video

<iframe width="560" height="315" src="https://www.youtube.com/embed/sKm2gfQ67GA?list=PLrUjPk-W0lafg86TqbD-bflgu1xSHiNwL" frameborder="0" allowfullscreen="allowfullscreen"></iframe> 

#### Conclusion



While finding Chuck Norris was kind of trivial, the concept behind it is very powerful. Think about a container that is restarting constantly in your infrastructure and is not even giving you the time to go there and inspect what’s going on. By the time you are able to do a `docker exec`, the container has already restarted ten times. With this, you are able to take a snapshot of your system and look at what happens inside your container on your own time. For more real world examples check out, [How we found a bug in Amazon ELB][2], [Fishing for Hackers][3], [Troubleshooting Cassandra Column Selection][4], or many of the other posts on our blog.



Stay tuned for more! Future exercises will include:



*   Unraveling Kubernetes messes: Finding a failing HTTP request, the respective TCP connection, and the Pod it originated from
*   Snooping tar activity
*   Troubleshooting container connectivity issues



<blockquote class="twitter-twee tw-align-center" data-lang="en">
  <p lang="en" dir="ltr">
    How to detect SSH attempts by Chuck Norris <a href="https://t.co/hUvqIK58Vw">https://t.co/hUvqIK58Vw</a>
  </p>— Sysdig (@sysdig) 
  
  <a href="https://twitter.com/sysdig/status/842442524325212161">March 16, 2017</a>
</blockquote>

<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script> 

If there are any examples with Kubernetes, Docker, Mesos, or anything going on in your system that you’d want us to do a post on. Let us know on <a href="https://twitter.com/sysdig" target="_blank">@sysdig</a> or our <a href="https://slack.sysdigrp2rs.wpengine.com/" target="_blank">Sysdig community Slack</a>!

 [1]: https://go.sysdigrp2rs.wpengine.com/chuck-norris
 [2]: https://sysdigrp2rs.wpengine.com/blog/amazon-elb-bug/
 [3]: https://sysdigrp2rs.wpengine.com/blog/fishing-for-hackers/
 [4]: https://sysdigrp2rs.wpengine.com/blog/column-selection-effects-query-performance/