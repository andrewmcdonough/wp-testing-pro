---
ID: 249
post_title: Announcing Sysdig 0.1.85
author: Gianluca Borello
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/announcing-sysdig-0-1-85/
published: true
post_date: 2014-07-09 09:15:01
---
This is mainly a bugfix release, targeting two rare but critical bugs that could cause a kernel panic when dealing with certain system call arguments. 
**The main new feature is sysdig completion for bash and zsh.**

What this means is that from now on you don't have to look at the manual constantly when using sysdig, and you can just press the TAB key any time you're struggling remembering the syntax, and the possible alternatives will come up, like in this bash example:

<pre>gianluca@sid:~$ sysdig --list[Tab]
--list  --list-chisels  --list-events</pre>

Or, even cooler, with zsh you can see a whole bunch of "dynamic" things:

<pre>gianluca@sid~% sysdig -c[Tab]
bottlenecks         -- Slowest system calls
echo_fds            -- Print the data read and written by processes.
fdbytes_by          -- I/O bytes, aggregated by an arbitrary filter field
fdcount_by          -- FD count, aggregated by an arbitrary filter field
fileslower          -- Trace slow file I/O
iobytes             -- Sum of I/O bytes on any type of FD
iobytes_file        -- Sum of file I/O bytes
iobytes_net         -- Show total network I/O bytes
list_login_shells   -- List the login shell IDs
netlower            -- Trace slow network I/0
proc_exec_time      -- Show process execution time
...</pre>

As you can see, the support for zsh is much nicer than the bash one, it automatically completes chisels and filter fields! We gladly welcome patches from the community to bring the bash completion at the same level!

In order to use this, if you install sysdig through our binary repository, the completion files are already installed under /usr/share/zsh/vendor-completions/\_sysdig and /etc/bash\_completion.d/sysdig, so the majority of the distributions will automatically use them by default. If you install sysdig from sources, "make install" will put them in /usr/local/share/zsh/vendor-completions/\_sysdig and /usr/local/etc/bash\_completion.d/sysdig, so you might have to tweak a bit your shell to get access to it, as usual.

## Resources

[Release details][1]

[Update instructions][2]

[Installation instructions][3]

[Source code][4]

## Support

Community support is available on the [sysdig mailing list][5].

Bugs and issues can be [submitted through github][6].

 [1]: https://github.com/draios/sysdig/releases
 [2]: https://github.com/draios/sysdig/wiki/Sysdig%20Update%20and%20Uninstall
 [3]: http://www.sysdig.org/install/
 [4]: https://github.com/draios/sysdig
 [5]: https://groups.google.com/forum/#!forum/sysdig
 [6]: https://github.com/draios/sysdig/issues?state=open