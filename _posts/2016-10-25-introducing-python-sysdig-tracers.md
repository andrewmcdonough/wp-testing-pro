---
ID: 3269
post_title: Introducing Python sysdig tracers
author: Jorge Salamero Sanz
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/introducing-python-sysdig-tracers/
published: true
post_date: 2016-10-25 10:57:21
---
Today we are opensourcing a python library to easy emit sysdig tracers. This allows you to quickly instrument python code and keep an eye on what your code does and how it performs. 
## Sysdig Tracers

Recently we have introduced in sysdig a new compelling feature: tracers. They allow you to trace everything with sysdig: method calls, API requests, specific pieces of code (spans), you name it!

To emit a tracer you need just to write a string to /dev/null with a pre-defined syntax and that’s it. It works for any kind of programming language, even from your bash/zsh shell :).

For more details check-out <a href="https://sysdigrp2rs.wpengine.com/blog/sysdig-tracers/" target="_blank">Introducing Sysdig Tracers: open source transaction tracing meets htop and strace.</a>

## Tracer Libraries for Node, Go, and Python

To make your life even simpler, we now have a few libraries to wrap up tracers. Two of these were created by TJ Holowaychuk (<a href="https://twitter.com/tjholowaychuk" target="_blank">@tjholowaychuk</a>): <a href="https://github.com/tj/node-trace" target="_blank">node-trace</a> and <a href="https://github.com/tj/go-trace" target="_blank">go-trace</a>. Our eternal thanks for his support and contributions. You need to follow TJ <a href="https://github.com/tj" target="_blank">on GitHub</a> too. Do it now!

Sysdig has also released a python library: <a href="https://github.com/draios/tracers-py" target="_blank">tracers-py</a>. Let’s have a look to a few little examples that shows off the library, and also what you can do with tracers.

## Tracers-py Our design goal is to make tracers as simple to use as possible. Besides writing something to a file is pretty easy, you need to know the exact syntax, do some lifting and so on. Our library is a simple wrapper to make tracers an integrated pythonic experience, they will be just one line of code away. 

Let’s see a little snippet of code to understand how it works:

<script src="https://gist.github.com/67f63ed2ab86de6015ba9526d008b024.js"></script> <noscript>
  <pre><code> File: sysdig_tracers_example1.py -------------------------------- from sysdig_tracers import Tracer import time while True: # here it is, it will emit a tracer when entering and exiting `with` block # tag parameter is optional, it will otherwise auto-detect it from your code # with pattern: filename.py:linenum(function_name) with Tracer(tag=&quot;hello_world&quot;): print &quot;Hello World&quot; print &quot;Hello World outside&quot; time.sleep(0.5) </code></pre>
</noscript>

And you’ll see on sysdig:

<pre># sysdig evt.type = tracer
115464 18:04:21.950697998 1 python (13735) &gt; tracer id=13735 tags=hello_world args=
115468 18:04:21.950773788 1 python (13735) &lt; tracer id=13735 tags=hello_world args=
115756 18:04:22.451915898 1 python (13735) &gt; tracer id=13735 tags=hello_world args=
115760 18:04:22.451971114 1 python (13735) &lt; tracer id=13735 tags=hello_world args=
116033 18:04:22.953642790 1 python (13735) &gt; tracer id=13735 tags=hello_world args=
</pre>

But you can leverage them to track just the system events related to that piece of code for example:

<pre># sysdig evtin.span.tags = hello_world
689 18:07:56.870876714 1 python (13737) &gt; write fd=1(&lt;f>/dev/pts/0) size=12
690 18:07:56.870886666 1 python (13737) &gt; switch next=12200 pgft_maj=0 pgft_min=1048 vm_size=25320 vm_rss=7560 vm_swap=0
692 18:07:56.870894988 1 python (13737) &lt; write res=12 data=Hello World.
693 18:07:56.870895614 1 python (13737) &gt; switch next=12200 pgft_maj=0 pgft_min=1048 vm_size=25320 vm_rss=7560 vm_swap=0
696 18:07:56.870902672 1 python (13737) &lt; tracer id=13737 tags=hello_world args=
&lt;/f></pre>

Remember you can use also csysdig, as another example here we are tracing a sample *worker* function:

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/10/sysdig_tracers_1.png"><img src="/wp-content/uploads/2016/10/sysdig_tracers_1-300x150.png" alt="csysdig opening file with tracers" width="600" height="300" class="alignnone size-medium wp-image-3271" /></a>

And we can also use drill down feature to see where the time is spent within it:

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/10/sysdig_tracers_2.png"><img src="/wp-content/uploads/2016/10/sysdig_tracers_2-300x150.png" alt="csysdig opening file with tracers" width="600" height="300" class="alignnone size-medium wp-image-3273" /></a>

## Advanced usage

The API is simple but can be used in many other ways, for example you can add custom arguments:

<script src="https://gist.github.com/fedd200dc5437325a49654e221a352e6.js"></script> <noscript>
  <pre><code> File: sysdig_tracers_example2.py -------------------------------- from sysdig_tracers import Tracer current_user = getUserID() with Tracer(enter_args={â€œuseridâ€: current_user} print &quot;Hello World&quot; </code></pre>
</noscript>

Or use it as a function decorator:

<script src="https://gist.github.com/45d10f742ab1212447649cd52646825c.js"></script> <noscript>
  <pre><code> File: sysdig_tracers_example3.py -------------------------------- from sysdig_tracers import Tracer @Tracer def myprinter(): print &quot;Hello World&quot; </code></pre>
</noscript>

You can also trace function arguments and return values, create nested spans and many other things. Check out our <a href="https://github.com/draios/tracers-py/tree/master/examples" target="_blank"> examples</a> directory for more.

## Recap

Our new python library makes easy to emit tracers and troubleshooting your code with sysdig. Go to our repository, try it out and let us now what do you think! Happy tracing!