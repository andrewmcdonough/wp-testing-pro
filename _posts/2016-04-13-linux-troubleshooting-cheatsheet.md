---
ID: 2508
post_title: 'Linux Troubleshooting Cheatsheet: strace, htop, lsof, tcpdump, iftop &#038; sysdig'
author: Phil Rzewski
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/linux-troubleshooting-cheatsheet/
published: true
post_date: 2016-04-13 03:20:08
---
This Sysdig cheatsheet is a great guide of command-lines linux admins can use to get insights into their servers. Whether you’ve been an admin for one month or 20 years you’ve definitely used one if not all of these tools to troubleshoot an issue. Because we love Sysdig (naturally!) we also included a translation for each of these common operations into the sysdig command line or csysdig.

Rather than attempt covering all options from manpages (which would have made for boring coverage of many esoteric, rarely-used switches), we’ve started from examples referenced at the most popular web pages you’d find when you search for terms like “strace examples”, “htop examples”, and so forth.

Do you have favorites that aren't listed here? Let us know and we’ll include them in future articles.

## strace

<span size="3" style="font-size: medium;">There’s one subtle difference between strace and sysdig that will be apparent in many of these side-by-side comparisons: Many of the simplest strace examples include command-lines that are executed and traced as a “one-shot” operation. On the other hand, Sysdig has a somewhat different philosophy, in that it either watches live events from afar as they happen, or analyzes capture data previously saved to a file. Thankfully, Sysdig's rich filtering options provide the knobs to watch for specific one-shot executions, as you’ll soon see.</span>

  
<div class="table-responsive" id="strace">
  <table class="table table-bordered ecosystem-table">
    <thead>
      <tr>
        <th class="text-center">
          Operation
        </th>
        
        <th class="text-center">
          strace
        </th>
        
        <th class="text-center">
          sysdig
        </th>
        
        <th class="text-center">
          Note
        </th>
      </tr>
    </thead>
    
    <tbody>
      <tr>
        <td>
          Trace the execution of a command
        </td>
        
        <td>
          <code>strace who </code>
        </td>
        
        <td>
          <code>sysdig proc.name=who </code>
        </td>
        
        <td>
          Whereas strace runs the who command shown here as a one-shot, Sysdig is watching for the execution of who. Use Sysdig’s filtering to further isolate a specific run, e.g.: <br /> <br /> <code>sysdig proc.name=who and proc.ppid=534</code> <br /> <br /> This watches for a who that’s about to be run in a shell that you’ve determined to have PID of 534.
        </td>
      </tr>
      
      <tr>
        <td>
          Trace only when certain/specific system calls are made
        </td>
        
        <td>
          <code>strace -e open who</code> <br /> <br /> <code>strace -e trace=open,read who</code>
        </td>
        
        <td>
          <code>sysdig evt.type=open and proc.name=who</code><br /> <br /> <code>sysdig "evt.type in (open,read) and proc.name=who"</code>
        </td>
        
        <td>
        </td>
      </tr>
      
      <tr>
        <td>
          Save a trace to a file
        </td>
        
        <td>
          <code>strace -o output.txt who </code>
        </td>
        
        <td>
          <code>sysdig -w output.scap proc.name=who </code>
        </td>
        
        <td>
          With strace, the file produced contains the same text you’d have viewed on the screen if run interactively. With Sysdig, you get a raw, re-usable capture file, such that you can view the text output with: <br /> <br /> <code>sysdig -r output.scap</code> <br /> <br /> You could also use this as the basis to apply filters or any other Sysdig functionality you want to apply as you revisit the original events.
        </td>
      </tr>
      
      <tr>
        <td>
          Watch a running process with PID=1363
        </td>
        
        <td>
          <code>strace -p 1363 </code>
        </td>
        
        <td>
          <code>sysdig proc.pid=1363 </code>
        </td>
        
        <td>
        </td>
      </tr>
      
      <tr>
        <td>
          Print a timestamp for each output line of the trace
        </td>
        
        <td>
          <code>strace -t who </code>
        </td>
        
        <td>
          <code>sysdig proc.name=who </code>
        </td>
        
        <td>
          Sysdig prints timestamps by default.
        </td>
      </tr>
      
      <tr>
        <td>
          Print relative time for system calls
        </td>
        
        <td>
          <code>strace -r who </code>
        </td>
        
        <td>
          <code>sysdig -tD proc.name=who </code>
        </td>
        
        <td>
          Sysdig offers several more ways to represent timestamps via the -t option.
        </td>
      </tr>
      
      <tr>
        <td>
          Generate batch statistics reports of system calls
        </td>
        
        <td>
          <code>strace -c who </code>
        </td>
        
        <td>
          <code>sysdig -w output.scap proc.name=who&lt;br /> # Now run the “who” separately</code> <br /> <br /> For one-shot batch text reports: <br /> <code>sysdig -r output.scap -c topscalls -c topscalls_time</code> <br /> <br /> Or for an interactive report that allows for further drill-down: <br /> <code>csysdig -r output.scap -v syscalls</code>
        </td>
        
        <td>
          Sysdig’s default behavior is more optimized for the case of presenting event data as it happens rather than “batch” reporting. This is why the Sysdig equivalent is done in two steps here.
        </td>
      </tr>
      
      <tr>
        <td>
          Generate live, per-second statistics reports of system calls for running process with PID=1363
        </td>
        
        <td>
          N/A
        </td>
        
        <td>
          <code>csysdig -v syscalls proc.pid=1363</code>
        </td>
        
        <td>
          While strace can show individual events as they happen live, or provide a single batch report for the execution of a command, csysdig’s views provide a unique ability to show live, periodic reports
        </td>
      </tr>
    </tbody>
  </table>
</div>

### htop

<span size="3" style="font-size: medium;">Since htop is a live, interactive, curses-style tool, we’ll compare it to the live, interactive, curses-style csysdig.</span>

<span size="3" style="font-size: medium;">For starters, both tools use the same approach of navigating the live table via Up/Down/Left/Right arrows and also PgUp/PgDn. For operations that affect a single process (killing, renicing, etc.) it is assumed you’ve used these controls to first highlight a particular process.</span>

  
<div class="table-responsive" id="htop">
  <table class="table table-bordered ecosystem-table">
    <thead>
      <tr>
        <th class="text-center" style="width: 25%;">
          Operation
        </th>
        
        <th class="text-center" style="width: 25%;">
          htop
        </th>
        
        <th class="text-center" style="width: 25%;">
          csysdig
        </th>
        
        <th class="text-center" style="width: 25%;">
          Note
        </th>
      </tr>
    </thead>
    
    <tbody>
      <tr>
        <td>
          Change sort order based on a column of the table
        </td>
        
        <td>
          Press <code>F6</code>, <code>&lt;</code>, or <code>&gt;</code> and then select a column by name, or <br /> Press <code>M</code>, <code>P</code>, or <code>T</code>to sort by Memory, Processor Usage, or Time <br /> Press <code>I</code> to invert the sort order
        </td>
        
        <td>
          Press <code>F9</code> or <code>&gt;</code> and then select a column by name, or<br /> <br /> Press <code>shift&lt;1-9&gt;<!--1-9--></code> to sort by any column 
          
          <code>n</code>, and press repeatedly to invert sort order, or<br /> <br /> Mouse-click on a column header
        </td>
        
        <td>
        </td>
      </tr>
      
      <tr>
        <td>
          Kill a process
        </td>
        
        <td>
          Press<code>F9</code> or <code>k</code>
        </td>
        
        <td>
          Press<code>k</code>
        </td>
        
        <td>
        </td>
      </tr>
      
      <tr>
        <td>
          Renice a process
        </td>
        
        <td>
          Press <code>F7</code> or<code> ]</code> to reduce the nice value by 1 <br /> Press <code>F8</code> or <code>[</code> to increase the nice value by 1
        </td>
        
        <td>
          Press <code>]</code> to reduce the nice value by 1 <br /> Press <code>[</code> to increase the nice value by 1
        </td>
        
        <td>
          This illustrates how easy it is to customize Sysdig. I noticed when first writing this article that csysdig was missing a couple minor features like this, so I used the opportunity to learn how easy it is to write/modify Chisels, then put up my improvements as a <a href="https://github.com/draios/sysdig/pull/560" target="_blank">Pull Request</a>. You can do the same!
        </td>
      </tr>
      
      <tr>
        <td>
          Display only processes started by a user named "phil"
        </td>
        
        <td>
          Press <code>u</code>, then <br /> Select the user name <code>phil</code> from the list
        </td>
        
        <td>
          Launch as:<br /> <code>csysdig user.name=phil</code> <br /> <br /> Or mouse-click <code>Filter:</code> from within <code>csysdig</code> at the top of the default Processes view, then append <code>and user.name=phil</code> to the current filter text
        </td>
        
        <td>
        </td>
      </tr>
      
      <tr>
        <td>
          Change the output refresh interval to once every 5 seconds
        </td>
        
        <td>
          Launch as:<br /> <code>htop -d 50</code>
        </td>
        
        <td>
          Launch as:<br /> <code>csysdig -d 5000</code>
        </td>
        
        <td>
          As you can see, htop works in units of tenths-of-a-second, while csysdig works in milliseconds.
        </td>
      </tr>
      
      <tr>
        <td>
          Start a system call trace on a process
        </td>
        
        <td>
          Press <code>s</code> to start an <code>strace</code>
        </td>
        
        <td>
          Press <code>F6</code> to start a <code>sysdig</code>
        </td>
        
        <td>
        </td>
      </tr>
      
      <tr>
        <td>
          List open files for a process
        </td>
        
        <td>
          Press <code>l</code> to run a one-time <code>lsof</code>
        </td>
        
        <td>
          Press <code>f</code> to run a one-time lsof Or to see real-time, updating reports of files/directories used by a process, drill down to a specific process by pressing <code>Enter</code>, then press <code>F2</code> and select a View such as <code>Files</code>, <code>File Opens List</code>, or <code>Directories</code>.
        </td>
        
        <td>
          See the Note above for “Renice a process” about how the one-time <code>lsof</code> was recently added as an enhancement.
        </td>
      </tr>
      
      <tr>
        <td>
          Follow a process, such that it remains highlighted even as its order in the list changes
        </td>
        
        <td>
          Press <code>F</code>
        </td>
        
        <td>
          Default behavior is to always follow the highlighted process
        </td>
        
        <td>
        </td>
      </tr>
    </tbody>
  </table>
</div>

### lsof

<div class="table-responsive" id="lsof">
  <table class="table table-bordered ecosystem-table">
    <thead>
      <tr>
        <th class="text-center">
          Operation
        </th>
        
        <th class="text-center">
          lsof
        </th>
        
        <th class="text-center">
          csysdig
        </th>
        
        <th class="text-center">
          Note
        </th>
      </tr>
    </thead>
    
    <tbody>
      <tr>
        <td>
          List all open files belonging to all active processes
        </td>
        
        <td>
          <code>lsof</code>
        </td>
        
        <td>
          <code>sysdig -c lsof</code>
        </td>
        
        <td>
        </td>
      </tr>
      
      <tr>
        <td>
          List processes that have opened the specific file /var/log/syslog
        </td>
        
        <td>
          <code>lsof /var/log/syslog</code>
        </td>
        
        <td>
          <code>sysdig -c lsof "fd.name=/var/log/syslog"</code>
        </td>
        
        <td>
        </td>
      </tr>
      
      <tr>
        <td>
          List processes that have opened files under the directory /var/log
        </td>
        
        <td>
          <code>lsof +d /var/log</code>
        </td>
        
        <td>
          <code>sysdig -c lsof "fd.directory=/var/log"</code>
        </td>
        
        <td>
        </td>
      </tr>
      
      <tr>
        <td>
          List files opened by processes named “sshd”
        </td>
        
        <td>
          <code>lsof -c sshd</code>
        </td>
        
        <td>
          <code>sysdig -c lsof "proc.name=sshd"</code>
        </td>
        
        <td>
        </td>
      </tr>
      
      <tr>
        <td>
          List files opened by a specific user named “phil”
        </td>
        
        <td>
          <code>lsof -u phil</code>
        </td>
        
        <td>
          <code>sysdig -c lsof "user.name=phil"</code>
        </td>
        
        <td>
        </td>
      </tr>
      
      <tr>
        <td>
          List files opened by everyone except for the user named “phil”
        </td>
        
        <td>
          <code>lsof -u ^phil</code>
        </td>
        
        <td>
          <code>sysdig -c lsof "user.name!=phil"</code>
        </td>
        
        <td>
        </td>
      </tr>
      
      <tr>
        <td>
          List all open files for a specific process with PID=1081
        </td>
        
        <td>
          <code>lsof -p 1081</code>
        </td>
        
        <td>
          <code>sysdig -c lsof "proc.pid=1081"</code>
        </td>
        
        <td>
        </td>
      </tr>
      
      <tr>
        <td>
          List all files opened by user "phil" or a process named "sshd" (OR logic)
        </td>
        
        <td>
          <code>lsof -u phil -c sshd</code>
        </td>
        
        <td>
          <code>sysdig -c lsof "'user.name=phil or proc.name=sshd'"</code>
        </td>
        
        <td>
          Note the use of two layers of quotes with the Sysdig filter.
        </td>
      </tr>
      
      <tr>
        <td>
          List all files opened by an "sshd" process for user "phil" (AND logic)
        </td>
        
        <td>
          <code>lsof -u phil -c sshd -a</code>
        </td>
        
        <td>
          <code>sysdig -c lsof "'user.name=phil and proc.name=sshd'"</code>
        </td>
        
        <td>
          Note the use of two layers of quotes with the Sysdig filter.
        </td>
      </tr>
      
      <tr>
        <td>
          Observe repeating reports of open files based on live activity
        </td>
        
        <td>
          Enable repeat mode with one of: <br /> <code>lsof -r</code> <br /> <code>lsof +r </code>
        </td>
        
        <td>
          Similar live data can be obtained with a live/interactive csysdig view, launched like so: <br /> <code>csysdig -v files</code> <br /> <code>csysdig -v file_opens</code>
        </td>
        
        <td>
        </td>
      </tr>
      
      <tr>
        <td>
          List all network connections
        </td>
        
        <td>
          <code>lsof -i</code>
        </td>
        
        <td>
          <code>sysdig -c lsof "fd.type=ipv4"</code>
        </td>
        
        <td>
        </td>
      </tr>
      
      <tr>
        <td>
          List network connections in use by a specific process with PID=1014
        </td>
        
        <td>
          <code>lsof -i -a -p 1014</code>
        </td>
        
        <td>
          <code>sysdig -c lsof "'fd.type=ipv4 and proc.pid=1014'"</code>
        </td>
        
        <td>
          Note the use of two layers of quotes with the Sysdig filter.
        </td>
      </tr>
      
      <tr>
        <td>
          List processes that are listening on port 22
        </td>
        
        <td>
          <code>lsof -i :22</code>
        </td>
        
        <td>
          <code>sysdig -c lsof "'fd.port=22 and fd.is_server=true'"</code>
        </td>
        
        <td>
          Note the use of two layers of quotes with the Sysdig filter.
        </td>
      </tr>
      
      <tr>
        <td>
          List all TCP or UDP connections
        </td>
        
        <td>
          <code>lsof -i tcp</code> <br /> <br /> <code>lsof -i udp</code>
        </td>
        
        <td>
          <code>sysdig -c lsof "fd.l4proto=tcp"</code> <br /> <br /> <code>sysdig -c lsof "fd.l4proto=udp"</code>
        </td>
        
        <td>
        </td>
      </tr>
    </tbody>
  </table>
</div>

### tcpdump

<span size="3" style="font-size: medium;">tcpdump is focused entirely on network traffic, while network traffic is only a subset of what Sysdig covers. Many tcpdump use cases involve filtering, and tcpdump uses network-specific <a href="https://en.wikipedia.org/wiki/Berkeley_Packet_Filter" target="_blank">BPF filters</a>, whereas Sysdig uses its own broader <a href="http://www.sysdig.org/wiki/sysdig-user-guide/#output-formatting" target="_blank">Sysdig filtering</a>. The two approaches look similar in many ways, but you’ll want to look at the docs for each side-by-side as you progress to more advanced filtering needs. Also, since in Linux <a href="https://en.wikipedia.org/wiki/Everything_is_a_file" target="_blank">everything is a file</a>, you’ll notice the Sysdig filtering examples below all leverage a “network-connections-via-file-descriptors” approach.</span>

  
<div class="table-responsive" id="tcpdump">
  <table class="table table-bordered ecosystem-table">
    <thead>
      <tr>
        <th class="text-center">
          Operation
        </th>
        
        <th class="text-center">
          tcpdump
        </th>
        
        <th class="text-center">
          csysdig
        </th>
        
        <th class="text-center">
          Note
        </th>
      </tr>
    </thead>
    
    <tbody>
      <tr>
        <td>
          Capture packets from a particular interface eth0 (192.168.10.119)
        </td>
        
        <td>
          <code>tcpdump -i eth0</code>
        </td>
        
        <td>
          <code>sysdig fd.ip=192.168.10.119</code>
        </td>
        
        <td>
          Sysdig does not currently have filtering based on named interfaces, but the equivalent via IP address is shown here.
        </td>
      </tr>
      
      <tr>
        <td>
          Capture only 100 packets
        </td>
        
        <td>
          <code>tcpdump -c 100</code>
        </td>
        
        <td>
          <code>sysdig -n 100 fd.type=ipv4</code>
        </td>
        
        <td>
        </td>
      </tr>
      
      <tr>
        <td>
          Display captured packets in ASCII
        </td>
        
        <td>
          <code>tcpdump -A</code>
        </td>
        
        <td>
          <code>sysdig -A fd.type=ipv4</code>
        </td>
        
        <td>
        </td>
      </tr>
      
      <tr>
        <td>
          Display captured packets in HEX and ASCII
        </td>
        
        <td>
          <code>tcpdump -XX</code>
        </td>
        
        <td>
          <code>sysdig -X fd.type=ipv4</code>
        </td>
        
        <td>
        </td>
      </tr>
      
      <tr>
        <td>
          Capture packet data, writing it into into a file
        </td>
        
        <td>
          <code>tcpdump -w saved.pcap</code>
        </td>
        
        <td>
          <code>sysdig -w saved.scap fd.type=ipv4</code>
        </td>
        
        <td>
          The Sysdig file format is capable of holding event data for much more than just network packets (e.g. system calls).
        </td>
      </tr>
      
      <tr>
        <td>
          Read back saved packet data from a file
        </td>
        
        <td>
          <code>tcpdump -r saved.pcap</code>
        </td>
        
        <td>
          <code>sysdig -r saved.scap</code>
        </td>
        
        <td>
        </td>
      </tr>
      
      <tr>
        <td>
          Capture only packets longer/smaller than 1024 bytes
        </td>
        
        <td>
          <code>tcpdump greater 1024</code><br /> <br /> <code>tcpdump less 1024</code>
        </td>
        
        <td>
          <code>sysdig "fd.type=ipv4 and evt.buflen &gt; 1024"</code> <br /> <br /> <code>sysdig "fd.type=ipv4 and evt.buflen &lt; 1024"</code>
        </td>
        
        <td>
          The <code>greater/less</code> options in tcpdump reference overall packet length whereas <code>evt.buflen</code> in Sysdig is relative to payload size.
        </td>
      </tr>
      
      <tr>
        <td>
          Capture only UDP or TCP packets
        </td>
        
        <td>
          <code>tcpdump udp</code><br /> <br /> <code>tcpdump tcp</code>
        </td>
        
        <td>
          <code>sysdig fd.l4proto=udp</code><br /> <br /> <code>sysdig fd.l4proto=tcp</code>
        </td>
        
        <td>
          Note that we don’t need to explicitly include <code>fd.type=ipv4</code> since we’re using other network-only filters here.
        </td>
      </tr>
      
      <tr>
        <td>
          Capture only packets going to/from a particular port
        </td>
        
        <td>
          <code>tcpdump port 22</code>
        </td>
        
        <td>
          <code>sysdig fd.port=22</code>
        </td>
        
        <td>
          Note that we don’t need to explicitly include <code>fd.type=ipv4</code> since we’re using other network-only filters here.
        </td>
      </tr>
      
      <tr>
        <td>
          Capture packets for a particular destination IP and port
        </td>
        
        <td>
          <code>tcpdump dst 54.165.81.189 and port 6666</code>
        </td>
        
        <td>
          <code>sysdig fd.rip=54.165.81.189 and fd.port=6666</code>
        </td>
        
        <td>
          Note that we don’t need to explicitly include <code>fd.type=ipv4</code> since we’re using other network-only filters here.
        </td>
      </tr>
    </tbody>
  </table>
</div>

### iftop

<span size="3" style="font-size: medium;">Since iftop is a live, interactive, curses-style tool, we’ll compare it to the live, interactive, curses-style csysdig. Also, like tcpdump, iftop uses BPF filters. See the previous intro to the section on tcpdump for more detail about filtering differences.</span>

  
<div class="table-responsive" id="iftop">
  <table class="table table-bordered ecosystem-table">
    <thead>
      <tr>
        <th class="text-center">
          Operation
        </th>
        
        <th class="text-center">
          iftop
        </th>
        
        <th class="text-center">
          csysdig
        </th>
        
        <th class="text-center">
          Note
        </th>
      </tr>
    </thead>
    
    <tbody>
      <tr>
        <td>
          Display a table of current bandwidth usage between pairs of hosts
        </td>
        
        <td>
          <code>iftop</code>
        </td>
        
        <td>
          Launch as: <br /> <code>csysdig -v connections</code><br /> <br /> Or press <code>F2</code> from within <code>csysdig</code> to change the View, then up-arrow to select <code>Connections</code>
        </td>
        
        <td>
          By default iftop watches just the first interface it finds, whereas by default csysdig watches traffic across the entire host.
        </td>
      </tr>
      
      <tr>
        <td>
          Turn on display of network ports
        </td>
        
        <td>
          Launch as: <br /> <code>iftop -P</code><br /> <br /> Or press <code>p</code> from within <code>iftop</code>
        </td>
        
        <td>
          Default behavior is to always display ports
        </td>
        
        <td>
        </td>
      </tr>
      
      <tr>
        <td>
          Observe traffic for just the eth0 interface (192.168.10.119)
        </td>
        
        <td>
          Launch as: <br /> <code>iftop -i eth0</code><br />
        </td>
        
        <td>
          Launch as: <br /> <code>csysdig -v connections fd.ip=192.168.10.119</code> <br /> <br /> Or mouse-click on <code>Filter:</code> from within <code>csysdig</code>, then append and <code>fd.ip=192.168.10.119</code> to the existing filter text
        </td>
        
        <td>
          sysdig/csysdig do not currently have filtering based on named interfaces, but the equivalent via IP address is shown here.
        </td>
      </tr>
      
      <tr>
        <td>
          Resolve DNS names
        </td>
        
        <td>
          Press <code>n</code> from within <code>iftop</code> to toggle resolution for all hosts shown
        </td>
        
        <td>
          Press <code>n</code> from within <code>csysdig</code> to run <code>nslookup</code> on the currently-highlighted remote host
        </td>
        
        <td>
        </td>
      </tr>
      
      <tr>
        <td>
          Change sort order based on a column of the table
        </td>
        
        <td>
          Press < to sort by source Press > to sort by destination
        </td>
        
        <td>
          Press <code>F9</code> or <code>&gt;</code> and then select a column by name, or <br /> <br /> Press <code>shift &lt;1-9&gt;<!--1-9--></code> to sort by any column 
          
          <code>n</code>, and press repeatedly to invert sort order, or<br /> <br /> Mouse-click on a column header
        </td>
        
        <td>
        </td>
      </tr>
      
      <tr>
        <td>
          Filter to show only traffic going to/from IP address 54.84.222.1
        </td>
        
        <td>
          Launch as:<br /> <code>iftop -f "host 54.84.222.1"</code>
        </td>
        
        <td>
          Launch as:<br /> <code>csysdig -v connections fd.ip=54.84.222.1</code> <br /> <br /> Or mouse-click on <code>Filter:</code> from within csysdig, then append <code>and fd.ip=54.84.22.1</code> to the existing filter text
        </td>
        
        <td>
        </td>
      </tr>
      
      <tr>
        <td>
          Pause the display
        </td>
        
        <td>
          Press <code>P</code>
        </td>
        
        <td>
          Press <code>p</code>
        </td>
        
        <td>
        </td>
      </tr>
      
      <tr>
        <td>
          Scroll the display
        </td>
        
        <td>
          Press <code>j</code> to scroll up <br /> Press <code>k</code> to scroll down
        </td>
        
        <td>
          Press <code>Up/Down/Left/Right</code> arrows or <code>PgUp/PgDn</code> to scroll through the table
        </td>
        
        <td>
          sysdig/csysdig go well beyond scrolling through a single-table, since you can drill down into the Connections View to see data in other groupings such as per-container or per-thread.
        </td>
      </tr>
    </tbody>
  </table>
</div>

<blockquote class="twitter-tweet tw-align-center" data-cards="hidden" data-lang="en">
  <p lang="en" dir="ltr">
    A linux troubleshooting cheatsheet: strace, htop, lsof, tcpdump, iftop & sysdig <a href="https://t.co/XeIeAwwj9i">https://t.co/XeIeAwwj9i</a>
  </p>— Sysdig (@sysdig) 
  
  <a href="https://twitter.com/sysdig/status/720619860036694016">April 14, 2016</a>
</blockquote>

<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script> 
## Sysdig CheatSheet Acknowledgements

The author would like to acknowledge <a href="http://www.thegeekstuff.com/" target="_blank">thegeekstuff.com</a>, as most of the example-filled articles used for the table above were found at their site.

**Would you like a downloadable PDF version of this cheatsheet?** [Grab it here][1].

 [1]: https://go.sysdigrp2rs.wpengine.com/troubleshooting-linux-cheatsheet