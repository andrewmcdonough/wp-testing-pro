---
ID: 2358
post_title: 50 shades of system calls
author: Loris Degioanni
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/50-shades-of-system-calls/
published: true
post_date: 2016-01-20 16:57:40
---
50 Shades of Systems Calls introduces Sysdig and csysdig system call profiling to monitor code performance inside Docker containers with spectrograms. This article has appeared on Hacker News on <a href="https://news.ycombinator.com/item?id=11159411" target="_blank">Feb 24, 2016</a> and <a href="https://news.ycombinator.com/item?id=12126852" target="_blank">Jul 19, 2016</a>. Also has been featured in Reddit <a href="https://www.reddit.com/r/sysadmin/comments/4774wn/50_shades_of_system_calls/" target="_blank">a</a> <a href="https://www.reddit.com/r/devel/comments/4tqfn8/50_shades_of_system_call/" target="_blank">few</a> <a href="https://www.reddit.com/r/programming/comments/4tnq4q/sysdig_50_shades_of_system_calls/" target="_blank">times</a>.

Sysdig spectrograms technology is used in <a href="https://www.sysdigrp2rs.wpengine.com/" target="_blank">Sysdig Monitor</a>, providing native container monitoring for Docker, Kubernetes or DC/OS Mesos.

Don’t want to read? Just watch this 30 second screen capture of spectrogram drill-down in csysdig.

<div style="text-align:center, clear:both">
  <iframe width="560" height="315" src="https://www.youtube.com/embed/bCS5NPuFYqg" frameborder="0" allowfullscreen></iframe>
</div>

## Background on the Sysdig spectrogram

Around a year ago I published a [Visualizing AWS Storage Latency with Real-Time Spectrograms][1] featuring a new Sysdig chisel called **spectrogram**. I suggest you go read it, but if you don’t have time here’s a recap of it works:

*   It captures every system call (e.g. `open()`, `close()`, `read()`, `write()`, `socket()`...) and it measures the call’s latency
*   Every 2 seconds it organizes the calls into buckets ranging from 1 nanosecond to 10 seconds
*   The spectrogram uses color to indicate how many calls are in each bucket 
    *   Black means no calls during the last sample
    *   Shades of green mean 0 to 100 calls
    *   Shades of yellow mean hundreds of calls
    *   Shades of red mean thousands of calls or more

As a consequence, the left side of the diagram tends to show stuff that is fast, while the right side shows stuff that is slow.

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/01/Spectrogram-picture-1.png"></a>[<img alt="csysdig spectrogram" class="alignnone" height="536" src="/wp-content/uploads/2016/01/Spectrogram-picture-1.png" width="750" />][2]

The spectrogram is great at showing patterns and outliers in the behavior of a system or application. But until now you are left to your own devices to troubleshoot *what is causing this spot in the chart?*

With a little more work, I was able to include it in csysdig, Sysdig’s command line ncurses UI, and also enhance it to be more useful.

## Interactive spectrograms in csysdig

Including the spectrogram in an interactive UI like csysdig makes it possible to enrich it in a couple of nice ways: arbitrary selections and drill-down into events.

The spectrogram can now be applied very easily to arbitrary selections in csysdig by clicking on `F12` (for a full spectrogram) or `SHIFT+F12` (for a spectrogram that includes file I/O activity only). As an example, you can easily view spectrogram information for a process, a directory, a file, a container or, thanks to csysdig’s [Kubernetes monitoring][3] with pods, services, deployments, etc. Having such a visualization just one keyboard stroke away makes it easy to use it for exploration and troubleshooting.

Even more interestingly, now you can use the mouse to drag over areas of a spectrogram and observe the events that belong to that area in the spectrogram.

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/01/Spectrogram-Picture-2.png"></a> [<img alt="csysdig spectrogram" class="alignnone" height="536" src= "/wp-content/uploads/2016/01/Spectrogram-Picture-2.png" width="750"/>][4]

A rectangular selection in the spectrogram isolates events that belong to a **latency range** (the width of the rectangle) in **a given interval in time** (the height of the rectangle).

Once a selection is made, you are brought to a page showing the events that respect the criteria of the selection.

Besides being positively fun, this drill down functionality tends to be very useful. Now your answer is just one mouse drag away.

Here’s an example.

## Writing To Disk in C, fwrite() vs write()

We are going to profile the following two simple C programs:

<div style="width: 49%; float: left; margin: 2px">
  <script src="https://gist.github.com/ce70defc28f3b0288166e27fd5b3a324.js"></script> <noscript>
    <pre><code>
File: write.c
-------------

#include &lt;stdlib.h&gt;
#include &lt;fcntl.h&gt;

int main()
{
    int j;
    char* buf = (char*)malloc(5000 * 1000);
    int fd; 

    fd = open("write.bin", O_CREAT | O_WRONLY);

    for(j = 0; j &lt; 5000; j++)
    {   
        write(fd, buf, j * 1000);
    }   
}

</code></pre>
  </noscript>
</div>

<div style="width: 49%; float: right; margin: 2px">
  <script src="https://gist.github.com/9ea8999a43e0d0a7f58535451cb5f977.js"></script> <noscript>
    <pre><code>
File: fwrite.c
--------------

#include &lt;stdio.h&gt;
#include &lt;stdlib.h&gt;

int main()
{
    int j;
    char* buf = (char*)malloc(5000 * 1000);

    FILE* f;
    f = fopen("write.bin", "w+");

    for(j = 0; j &lt; 5000; j++)
    {   
        fwrite(buf, j * 1000, 1, f); 
    }   
}
</code></pre>
  </noscript>
</div>

<p style="clear: both">
  They both do exactly the same things: they perform 5000 writes of increasing sizes to disk. The first one (on the left) works at the file descriptor level, while the second one uses the C standard I/O library.
</p>

Let’s take a look at the spectrogram of the first program:

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/01/spectrogram-picture-3.png"></a>[<img alt="csysdig spectrogram" class="alignnone" height="536" src= "/wp-content/uploads/2016/01/spectrogram-picture-3.png" width="750"/>][5]

As expected, latencies increase with write buffer sizes, at a pretty regular pace. If we observe the size of the disk writes by drilling down from any portion of the chart, it’s easy to see that they correspond to the ones that the program specifies.

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/01/spectrogram-terminal-selection-1.png"></a>[<img alt="csysdig terminal" class="alignnone" height="536" src= "/wp-content/uploads/2016/01/spectrogram-terminal-selection-1.png" width="750"/>][6]

The three isolated dots on the right of the spectrogram are slow writes that take between half a second and a second, likely because of OS cache flushes.

Now let’s looks at the spectrogram of the second program:

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/01/spectrogram-picture-4.png"></a>[<img alt="csysdig spectrogram" class="alignnone" height="536" src="/wp-content/uploads/2016/01/spectrogram-picture-4.png" width="750" />][7]

In this case, we clearly see a bimodal behavior, with an activity area in the tens of microseconds and another one between 1 and 30 milliseconds. What’s going on?

The right side contains a bunch of increasing size writes that sort of match the buffer size specified by the user in the call to `fwrite()`, but just a bit smaller. If we select an area in the left side of the chart, on the other hand, we can clearly see that these are all 4096 bytes disk writes.

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/01/Spectrogram-terminal-2.png"></a>[<img alt="csysdig terminal" class="alignnone" height="536" src="/wp-content/uploads/2016/01/Spectrogram-terminal-2.png" width="750" />][8]

These writes are fast because they are smaller. This bimodal behavior is caused by the additional layer of buffering introduced by `fwrite()`. Data is stored by `fwrite()` before being flushed to disk, the first 4KB are written right away and then the rest of the buffer is flushed in (typically) a single second write.

Which way is the best one? Buffered or unbuffered?

Well, as usual it depends. In the case above, with big and sequential writes, avoiding the additional buffering offers more predictable latency and slightly better performance. On the other hand, check out what happens when we do many tiny 1-byte writes:

<div style="width: 49%; float: right; margin: 2px">
  <script src="https://gist.github.com/50d001f1b15fc7b7f5aaf5824dab6665.js"></script> <noscript>
    <pre><code>
File: open.c
------------

#include &lt;stdlib.h&gt;
#include &lt;fcntl.h&gt;

int main()
{
    int j;
    char buf[1];
    int fd; 

    fd = open("write.bin", O_CREAT | O_WRONLY);

    for(j = 0; j &lt; 5000000; j++)
    {   
        write(fd, buf, 1); 
    }   
}
</code></pre>
  </noscript>
</div>

<div style="width: 49%; float: right; margin: 2px">
  <script src="https://gist.github.com/62adf90dd4ad5e0110e1ccacc7add6d4.js"></script> <noscript>
    <pre><code>
File: fopen.c
-------------

#include &lt;stdio.h&gt;
#include &lt;stdlib.h&gt;

int main()
{
    int j;
    char buf[1];
    FILE* f;

    f = fopen("write.bin", "w+");

    for(j = 0; j &lt; 5000000; j++)
    {   
        fwrite(buf, 1, 1, f); 
    }   
}
</code></pre>
  </noscript>
</div>

<p style="clear: both">
  This is the spectrum of the left program:
</p>

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/01/Spectrogram-Picture-5.png"></a>[<img alt="csysdig spectrogram" class="alignnone" height="536" src="/wp-content/uploads/2016/01/Spectrogram-Picture-5.png" width="750" />][9]

As you can see, **a lot** write system calls cause the program to take tens of seconds to complete, and generate quite a bit of system stress:

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/01/Spectrogram-picture-6.png"></a>[<img alt="csysdig terminal" class="alignnone" height="536" src="/wp-content/uploads/2016/01/Spectrogram-picture-6.png" width="750" />][10]

This, on the other hand, is the spectrum of the second program:

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/01/Spectrogram-terminal-6.png"></a>[<img alt="csysdig spectrogram" class="alignnone" height="536" src="/wp-content/uploads/2016/01/Spectrogram-terminal-6.png" width="750" />][11]

Pretty short, eh? Everything is coalesced into few 4096KB buffers, the number of system calls is very low and the program complete in less than half a second.

<a data-rel="lightbox-0" href="/wp-content/uploads/2016/01/Spectrogram-terminal-4.png"></a>[<img alt="csysdig terminal" class="alignnone" height="536" src="/wp-content/uploads/2016/01/Spectrogram-terminal-4.png" width="750" />][12]

## Conclusions

Decisions that can seem negligible, like choosing which API you use when writing to disk, can have a big impact on the performance of your application.

The complexity of modern systems often makes it hard to determine what the implications of these decisions are. Measuring things tends to be extremely important, especially when paired with the right visualizations and the ability to drill down and investigate.

Luckily, sysdig and csysdig are very [easy to install][13] and can be a valuable aid.

<blockquote class="twitter-tweet tw-align-center" data-lang="en">
  <p lang="en" dir="ltr">
    50 Shades of System Calls <a href="https://t.co/uNXb9XbhMz">https://t.co/uNXb9XbhMz</a> <a href="https://t.co/8cKgn9oCKo">pic.twitter.com/8cKgn9oCKo</a>
  </p>— Sysdig (@sysdig) 
  
  <a href="https://twitter.com/sysdig/status/702181265919909889">February 23, 2016</a>
</blockquote>

<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

 [1]: /blog/aws-storage-latency-sysdig-spectrogram/
 [2]: /wp-content/uploads/2016/01/Spectrogram-picture-1.png
 [3]: /blog/digging-into-kubernetes-with-sysdig/
 [4]: /wp-content/uploads/2016/01/Spectrogram-Picture-2.png
 [5]: /wp-content/uploads/2016/01/spectrogram-picture-3.png
 [6]: /wp-content/uploads/2016/01/spectrogram-terminal-selection-1.png
 [7]: /wp-content/uploads/2016/01/spectrogram-picture-4.png
 [8]: /wp-content/uploads/2016/01/Spectrogram-terminal-2.png
 [9]: /wp-content/uploads/2016/01/Spectrogram-Picture-5.png
 [10]: /wp-content/uploads/2016/01/Spectrogram-picture-6.png
 [11]: /wp-content/uploads/2016/01/Spectrogram-terminal-6.png
 [12]: /wp-content/uploads/2016/01/Spectrogram-terminal-4.png
 [13]: http://www.sysdig.org/install/