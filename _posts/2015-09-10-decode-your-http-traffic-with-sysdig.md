---
ID: 2045
post_title: Decode Your HTTP Traffic with sysdig
author: Loris Degioanni
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/decode-your-http-traffic-with-sysdig/
published: true
post_date: 2015-09-10 07:00:48
---
Sysdig goes application aware! Recently we’ve been adding little useful features to sysdig at a good pace, and one that is worth mentioning is HTTP decoding. 

Starting with <a href="https://sysdigrp2rs.wpengine.com/announcing-sysdig-0-1-103/" target="_blank">sysdig 0.1.103</a>, we have added two chisels, `httplog` and `httptop`, which provide insights into all HTTP traffic flowing on your server. They list the requests sent and received, and they can be used to find out what’s exactly hitting a web server. This blog post presents them and show a couple of usage tricks to get the best out of them. 

But first, let’s take a step back and talk about echoing network connections in sysdig. 

## The Old Way: The echo_fds Chisel 

For quite a bit of time, sysdig has offered a chisel to inspect the activity of a given set of file descriptors. It’s called `echo_fds`, and you can easily use it to troubleshoot network connections using the appropriate filter, for example: 
<pre># sysdig -A -pc -c echo_fds fd.port=80

<span style="color: red;">------ Read 81B from  [http_server_1] [70d6e2d0d30c]  172.17.2.110:51742->172.17.2.107:4567 (ruby)

GET /slow/1 HTTP/1.1
Host: server:4567
User-Agent: curl/7.42.1
Accept: */*
</span>

<span style="color: blue;">------ Write 282B to  [http_server_1] [70d6e2d0d30c]  172.17.2.117:38503->172.17.2.107:4567 (ruby)

HTTP/1.1 200 OK
Content-Type: text/html;charset=utf-8
Content-Length: 12
X-X
</span>
</pre>

`Echo_fds` has the nice benefit of being able to display network connections established by any container. And, of course, its functionality is nicely integrated into the <a href="https://www.youtube.com/watch?v=UJ4wVrbP-Q8" target="_blank">csysdig curses interface</a>. 

## The New Way: the httplog and httptop Chisels 

`Httplog` and `httptop` go one step further by implementing some basic decoding of the raw data that echo_fds shows. In particular, if the connection is carrying HTTP data, they extract information like the URL and the response time. You can think about this as a simple version of <a href="https://github.com/lebinh/ngxtop" target="_blank">ngxtop</a>, but web server agnostic and able to see inside containers. `Httplog` will print information about every request, in a way similar to a log: 
<pre># sysdig -pc -c httplog 
2015-08-10 09:15:54.159173634 http_server_1 &lt; method=GET url=server:4567/slow/3 response_code=200 latency=328ms size=12B
2015-08-10 09:15:54.166230840 http_server_1 &lt; method=GET url=server:4567/slow/1 response_code=200 latency=130ms size=12B
2015-08-10 09:15:54.166256264 http_client_5 > method=GET url=server:4567/slow/1 response_code=200 latency=158ms size=12B
2015-08-10 09:15:54.166969191 http_client_1 > method=GET url=server:4567/slow/1 response_code=200 latency=159ms size=12B
2015-08-10 09:15:54.167024171 http_server_1 &lt; method=GET url=server:4567/slow/1 response_code=200 latency=130ms size=12B
</pre></pre>

`Httptop`, on the other hand, offers a \`top-like\` view of all the HTTP transactions: 
<pre># sysdig -pc -c httptop 
ncalls          method          url
-------------------------------------------------------------------
182             GET             server:4567/slow/2
182             GET             server:4567/slow/3
171             GET             server:4567/slow/1
</pre>

## Filtering Fun 

As usual, leveraging sysdig’s filtering engine in conjunction to chisels can make your experience more fun and rewarding. 

For example, you can observe the web requests of a particular container: 
<pre># sysdig -pc -c httplog container.name=wordpress
</pre>

or the ones of that were not served by an expected process: 
<pre># sysdig -pc -c httplog “process.name!=httpd”
</pre>

or that are coming from a specific client: 
<pre># sysdig -pc -c httplog fd.cip=192.168.0.1
</pre>

## Wrapping Up 

The `httplog` and `httptop` chisels expand sysdig’s swiss army knife capabilities, and are great to keep an eye on your containerized web servers. And if you need this functionality (and much more!) but for all the machines and containers in your infrastructure, <a href="https://sysdigrp2rs.wpengine.com/landing-page/?utm_source=web&utm_medium=blog&utm_campaign=httpchisel091015" target="_blank">try Sysdig Cloud</a> for free for 14 days.