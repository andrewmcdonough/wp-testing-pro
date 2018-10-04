---
ID: 2891
post_title: System profiling for lazy developers
author: Loris Degioanni
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/system-profiling-lazy-developers/
published: true
post_date: 2016-08-04 11:41:17
---
Measuring latency within my code is something that I do very very often. Occasionally I resort to tools like profilers to help me out but, honestly, most of the time I just put timers in my code and print the results to the console or a log file. The reasons are:

*   Running a profiler requires quite a bit of setup, which often is not justified or I’m too lazy to do
*   A profiler tends to give me a very noisy output, like the full execution stack, while I usually only want to focus on a few specific portion of code

I bet that this sounds pretty familiar to you. And I bet that, if you’re lazy like I am, you’re going to be interested in how to profile your software and systems with less effort. Read on!

We are going to learn how to use a functionality recently added to sysdig, [tracers][1], to easily take measurements inside your code and then analyze them. To ground the story, I’ll use a real use case: Is the *aggregate* functionality introduced in MongoDB 2.2 faster than doing a traditional *map-reduce* call? If yes, how much better is it?

### Setup First of all, I need to deploy Mongo. Docker is my friend here: 

<pre>$&gt; docker run --name mongotest -d mongo </pre> Now I need some data for the queries. I can create it using the ruby 

[Faker][2] library. For example, this scripts creates 30,000 simulated customer entries: 
<pre><span style="color: #0000ff;">require</span> <span style="color: #008000;">'faker'</span>
<span style="color: #0000ff;">require</span> <span style="color: #008000;">'mongo'</span>

include <span style="color: #800080;">Mongo</span>

client = <span style="color: #800080;">MongoClient</span>.<span style="color: #0000ff;">new</span>(ENV[<span style="color: #008000;">'MONGODB'</span>], 27017)
db = client[<span style="color: #008000;">"test"</span>]
collection = db[<span style="color: #008000;">"customers"</span>]

<span style="color: #333399;">30000.times</span> <span style="color: #0000ff;">do</span>
	collection.insert({
			:first_name =&gt; <span style="color: #800080;">Faker::Name</span>.first_name,
			:last_name =&gt; <span style="color: #800080;">Faker::Name</span>.last_name,
			:city =&gt; <span style="color: #800080;">Faker::Address</span>.city,
			:country_code =&gt; <span style="color: #800080;">Faker::Address</span>.country_code,
			:orders_count =&gt; <span style="color: #800080;">Random</span>.rand(<span style="color: #333399;">10</span>)+<span style="color: #333399;">1</span>
		})
<span style="color: #0000ff;">end
</span></pre>

Now let’s execute the script:

<pre>$&gt; MONGODB=&lt;server_address> ruby filldata.rb&lt;/server_address></pre>

And voila, we have a MongoDB backend with a bunch of data to play with!

### Instrumentation

Now I’m going to create two scripts that each get the sum of the orders grouped by country code. However, the scripts leverage different underlying MongoDB facilities. The first one uses `map_reduce:```

<pre><span style="color: #0000ff;">require</span> <span style="color: #008000;">'mongo'</span>
include <span style="color: #800080;">Mongo</span>

$stdout.sync = <span style="color: #0000ff;">true</span>

client = <span style="color: #800080;">MongoClient</span>.<span style="color: #0000ff;">new</span>(ENV[<span style="color: #008000;">'MONGODB'</span>], 27017)
db = client[<span style="color: #008000;">"test"</span>]
collection = db[<span style="color: #008000;">"customers"</span>]

loop <span style="color: #0000ff;">do</span>
	<span style="color: #0000ff;">print</span> <span style="color: #008000;">"&gt;:t:map-reduce::\n"</span> <span style="color: #800000;"># Mark the beginning of the query</span>

	collection.map_reduce(<span style="color: #008000;">"function() { emit(this.country_code, this.orders_count) }",
	          "function(key,values) { return Array.sum(values) }"</span>,  { :<span style="color: #0000ff;">out</span> =&gt; { :<span style="color: #0000ff;">inline</span> =&gt; <span style="color: #0000ff;">true</span> }, :raw =&gt; <span style="color: #0000ff;">true</span>});

	<span style="color: #0000ff;">print</span> <span style="color: #008000;">"&lt;:t:map-reduce::\n"</span> <span style="color: #800000;"># Mark the end of the query</span>
<span style="color: #0000ff;">end
</span></pre>

While the second one uses aggregate:

<pre><span style="color: #0000ff;">require</span> <span style="color: #008000;">'mongo'</span>
include <span style="color: #800080;">Mongo</span>

$stdout.sync = <span style="color: #0000ff;">true</span>

client = <span style="color: #800080;">MongoClient</span>.<span style="color: #0000ff;">new</span>(ENV[<span style="color: #008000;">'MONGODB'</span>], 27017)
db = client[<span style="color: #008000;">"test"</span>]
collection = db[<span style="color: #008000;">"customers"</span>]

loop <span style="color: #0000ff;">do</span>
	<span style="color: #0000ff;">print</span> <span style="color: #008000;">"&gt;:t:aggregate::\n"</span> <span style="color: #800000;"># Mark the beginning of the query</span>

	collection.aggregate( [
	                    { <span style="color: #008000;">"$match"</span> =&gt; {}},
	                    { <span style="color: #008000;">"$group"</span> =&gt; {
	                                  <span style="color: #008000;">"_id"</span> =&gt; <span style="color: #008000;">"$country_code"</span>,
	                                  <span style="color: #008000;">"value"</span> =&gt; { <span style="color: #008000;">"$sum"</span> =&gt; <span style="color: #008000;">"$orders_count"</span> }
	                                }
	                   }
	           ])

	<span style="color: #0000ff;">print</span> <span style="color: #008000;">"&lt;:t:aggregate::\n"</span> <span style="color: #800000;"># Mark the end of the query</span>
<span style="color: #0000ff;">end
</span></pre>

Notice anything particular in these scripts? Yes, those prints in the code are sysdig tracers:

<pre><span style="color: #0000ff;">print</span> <span style="color: #008000;">"&lt;:t:aggregate::\n"</span> <span style="color: #800000;"># Mark the end of the query</span></pre>

In this case, tracers do the following:

*   They mark the beginning and the end of a span. **>** is the beginning, and **<** is the end
*   They give it a unique ID.**t** in this case means “use the thread ID as the span ID”, which is an easy way to get a unique number across executions
*   They also give it a name “map-reduce” is the name in the first script, “aggregate” in the second one.

You can read the [tracers manual][1] for the details but, as you can see, it’s pretty straightforward.

When you write these strings to */dev/null* and sysdig is running, sysdig captures them, decodes them and does the measurement magic for you.

So let’s run the scripts and redirect their output to /dev/null:

<pre>$&gt; MONGODB=&lt;server_address> ruby query_agg.rb &gt; /dev/null &
$&gt; MONGODB=&lt;/server_address>&lt;server_address> ruby query_mr.rb &gt; /dev/null &
&lt;/server_address></pre>

### Timing Analysis

Sysdig lets me run my analysis live, as the scripts are running. This time however, I’m going to take a capture file:

<pre>$&gt; sudo sysdig -w mongo.scap</pre>

[Download the capture file][3] and you will be able to practice what I describe in this blog post on your machine.

Let’s open the capture file with Csysdig:

<pre>$&gt; csysdig -r mongo.scap</pre>

Let’s take a high level look at the latencies, by selecting the *Traces Summary* view (F2 → Traces Summary):

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/08/animation.png"></a>[<img alt="Traces Summary" class="alignnone" height="536" src="/wp-content/uploads/2016/08/animation.png" width="750" />][4] 

This gives us the number of hits and timing information for the two types of queries that we instrumented. It’s evident that aggregate is much faster, more than 6 times on average.

Now let’s switch to a spectrogram view (F2 → Traces Spectrogram):

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/08/pectro.png"></a>[<img alt="Sysdig Spectrogram" class="alignnone" height="536" src="/wp-content/uploads/2016/08/pectro.png" width="750" />][5] 

[Spectrograms][6] allow us to visualize the latencies of almost anything. In sysdig they are often used for measuring system calls, and now we can use them for trace spans as well.What we see is two clear latency bands, one in the tens of milliseconds and one in the hundreds of milliseconds. This is consistent with the output of the Traces Summary view, so my thesis is that the left band is made by aggregate requests, while the right band is all map-reduce requests. Proving the thesis is just a mouse click away. The Csysdig spectrogram allows me to make a selection and drill down to see inside it. So let’s drill down into the left band:

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/08/spectro_sel_agg.png"></a>[<img alt="Sysdig Spectrogram" class="alignnone" height="536" src="/wp-content/uploads/2016/08/spectro_sel_agg.png" width="750" />][7] 

The result is shown in this picture and it’s exactly what I expected:

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/08/spectro_dd_agg.png"></a>[<img alt="Sysdig Spectrogram" class="alignnone" height="536" src="/wp-content/uploads/2016/08/spectro_dd_agg.png" width="750" />][8] 

The right band contains only map-reduce requests, and each of them takes much longer:

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/08/spectro_dd_mr.png"></a>[<img alt="Sysdig Spectrogram" class="alignnone" height="536" src="/wp-content/uploads/2016/08/spectro_dd_mr.png" width="750" />][9] 

### Span-Based Resource Monitoring

Once our app is instrumented with sysdig tracers, we can leverage full sysdig functionality in a span-aware way. For example, let’s compare the network bandwidth utilization of the two query options. There are several ways to do it with sysdig, but this time I’m going to use the echo_fd [chisel][10] and **just filter I/O from the map-reduce spans:**

<pre>$&gt; sysdig -r mongo.scap -c echo_fds evtin.span.tags=map-reduce

<span style="color: #0000ff;">------ Write 234B to   172.17.42.1:33431-&gt;172.17.0.28:27017 (ruby)
....m...............test.$cmd..............mapreduce.....customers..map.G...:...</span>
<span style="color: #800000;">------ Read 16B from   172.17.42.1:33431-&gt;172.17.0.28:27017 (ruby)
.$......m.......
------ Read 20B from   172.17.42.1:33431-&gt;172.17.0.28:27017 (ruby)
....................
------ Read 4B from   172.17.42.1:33431-&gt;172.17.0.28:27017 (ruby)
.#..
------ Read 8.99KB from   172.17.42.1:33431-&gt;172.17.0.28:27017 (ruby)
.results..#...0. ...._id.....AD..value........@..1. ...._id.....AE..value.......

</span></pre>

The snippet above isolates a single query. We can see that there’s a 191 byte request buffer and then another 4 buffers sent back from Mongo, for a total of 9480 bytes.

Now let’s try with aggregate:

<pre>$&gt; sysdig -r mongo.scap -c echo_fds evtin.span.tags=aggregate

<span style="color: #0000ff;">------ Write 184B to   172.17.42.1:33430-&gt;172.17.0.28:27017 (ruby)
....+...............test.$cmd..............aggregate.....customers..pipeline.q..</span>
<span style="color: #800000;">------ Read 16B from   172.17.42.1:33430-&gt;172.17.0.28:27017 (ruby)
.&......+.......
------ Read 20B from   172.17.42.1:33430-&gt;172.17.0.28:27017 (ruby)
....................
------ Read 4B from   172.17.42.1:33430-&gt;172.17.0.28:27017 (ruby)
.&..
------ Read 7.96KB from   172.17.42.1:33430-&gt;172.17.0.28:27017 (ruby)
.waitedMS..........result..&...0.#...._id.....ET..orders_count.......1.#...._id.

</span></pre>

Quite a bit smaller! A total of 8375 bytes, or around 12% better.

### Conclusion

The first, obvious conclusion is: laziness pays off! [Tracers][1] are an easier method of profiling just about anything: methods in your software, file access, network transactions...anything. The second conclusion is: use aggregate and not map-reduce in your mongo queries. It will be much faster and save you network bandwidth.

I hope that the measuring techniques explained in this blog post can be a useful addition to your toolbelt. We designed them to be as frictionless as possible but at the same time support common development use cases, especially in containerized environments.

The whole stack described in the post is open source and very easy to install. Give it a try and let us you know what you think.

 [1]: https://github.com/draios/sysdig/wiki/Tracers
 [2]: https://github.com/stympy/faker
 [3]: https://dl.dropboxusercontent.com/u/16649518/mongo.scap
 [4]: /wp-content/uploads/2016/08/animation.png
 [5]: /wp-content/uploads/2016/08/pectro.png
 [6]: https://sysdigrp2rs.wpengine.com/blog/50-shades-of-system-calls/
 [7]: /wp-content/uploads/2016/08/spectro_sel_agg.png
 [8]: /wp-content/uploads/2016/08/spectro_dd_agg.png
 [9]: /wp-content/uploads/2016/08/spectro_dd_mr.png
 [10]: https://github.com/draios/sysdig/wiki/Chisels-Overview