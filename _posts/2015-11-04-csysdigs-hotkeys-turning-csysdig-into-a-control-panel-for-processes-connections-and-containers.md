---
ID: 2200
post_title: 'csysdig’s Hotkeys: turning csysdig into a control panel for processes, connections and containers'
author: Loris Degioanni
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/csysdigs-hotkeys-turning-csysdig-into-a-control-panel-for-processes-connections-and-containers/
published: true
post_date: 2015-11-04 09:00:03
---
<a href="/wp-content/uploads/2015/10/Recording-149.gif" data-rel="lightbox-0" title><img src="/wp-content/uploads/2015/10/Recording-149.gif" /> </a> 

<p align="center">
  <i>jumping into a container from csysdig</i>
</p>

If you are not familiar with it, csysdig is a ncurses user interface for exploring your Linux system, Docker containers, and other information gathered by sysdig. <a href="https://www.youtube.com/watch?v=UJ4wVrbP-Q8" target="_blank">This video</a> offers a quick introduction. 

Up until now, csysdig has been entirely oriented around exploring information... while leaving you on your own take actions based on any insights you have. But, with the latest release, we’ve introduced an exciting new feature in csysdig: the ability to easily take actions in your environment, directly from the csysdig interface! This includes common actions like killing processes and attaching to containers, but also covers custom actions of your choice. You can even trigger these actions with simple mappings to keyboard keys. 

This new feature fits in pretty seamlessly with the existing workflow in csysdig. One of the most popular ways to interface with csysdig is by drilling down into selections: once you highlight an item with the keyboard or with the mouse, you can click enter to explore it, or F5 to view its I/O, or F6 to display the system calls it performs. With the new release, you can now associate arbitrary keys to execute commands in the context of your selection. Let me give you some examples. 

## Example Actions for Processes

After selecting a process, you will be able to: 
*   send it a kill signal (k) or a kill -9 signal (9)
*   generate a core dump (c)
*   attach gdb to the process (g)
*   see the process’ ltrace output (l)
*   print the process’ stack (s)

## Example Actions for Network Connections

These are the actions that you can perform on an entry in the connections view: 
*   run a tcpdump capture for the connection (c), for the local IP (l) and for the remote IP (r)
*   ping the remote connection endpoint (p)
*   traceroute the remote connection endpoint (t)
*   do an nslookup of the remote IP (n)

## Example Actions for Containers

csysdig actions become really interesting when used in conjunction with Docker containers. The csysdig user interface becomes a quick and visual alternative to the Docker command line. For example, you can: 
*   attach to the selected container (a)
*   open a bash shell in the selected container (b)
*   get the container’s image history (h)
*   kill (k), stop (s) or pause (z) the container
*   follow the container logs (f)

## How to Find Out Which Hotkeys Are Available

Each view has its own hot keys, designed for the data that the view shows. You have two ways to see what they are: 
*   clicking on F8 opens a sidebar with the list of hotkeys and lets you pick one by scrolling up and down
*   clicking on F7 shows the help page for the view. This page contains the list of hotkeys, including the specific command line executed when the hotkey is pressed

## Customizing csysdig With Your Own Hotkeys

Want to add your own hotkey? For example you could add a hotkey to delete a file when selected. Nothing easier! 

The nice thing about csysdig is that it’s fully customizable through simple Lua scripts. In particular, csysdig’s views are defined in /usr/share/sysdig/chisels. Adding your own hotkey and command is just a matter of editing the actions section. The syntax is described <a href="https://github.com/draios/sysdig/wiki/Csysdig-View-Format-Reference#action-fields" target="_blank">here</a>. 

## Conclusions

Hotkeys are a small addition to sysdig, but as a user I think they have a big impact on its usefulness. In particular, I find the Docker command line shortcuts really useful. Being able to connect to container using a lot of CPU by pressing a key instead of having to copy and paste its ID saves a lot of time and prevents mistakes. 

As mentioned, actions are completely configurable. We tried to come up with a useful default set, but if you have an interesting suggestion, please shout out in the comments or send a pull request, and we'll make sure to include it!