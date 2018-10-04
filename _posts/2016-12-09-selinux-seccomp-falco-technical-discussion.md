---
ID: 3415
post_title: 'SELinux, Seccomp, Sysdig Falco, and you: A technical discussion'
author: Mark Stemm
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/selinux-seccomp-falco-technical-discussion/
published: true
post_date: 2016-12-09 01:24:16
---
One of the questions we often get when we talk about <a href="http://www.sysdig.org/falco/" target="_blank" rel="noopener">Sysdig Falco</a> is *How does it compare to other tools like SELinux, AppArmor, Auditd, etc. that also have security policies?* To help answer some of those questions, we thought we'd present a summary of other related security products and how they compare to <a href="http://www.sysdig.org/falco/" target="_blank" rel="noopener">Sysdig Falco</a>.

Specifically, we’ll look at the following tools:

*   [Basic sandboxing: seccomp][1]
*   [Sandboxing with policies: seccomp-bpf][2]
*   [Mandatory access control systems: SELinux, AppArmor][3]
*   [System auditing: Auditd][4]
*   [Behavioral monitoring: Falco][5]

Overall, these products can be grouped into ones focused on *enforcement* vs *auditing*. Both groups define a policy that describes the allowed or disallowed behavior for a process, in terms of system calls, their arguments, and host resources accessed.

Enforcement tools use the policy to change the behavior of a process by preventing system calls from succeeding, or in some cases, killing the process. Seccomp, seccomp-bpf, SELinux, and AppArmor are examples of enforcement tools.

Auditing tools use the policy to monitor the behavior of a process and notify when its behavior steps outside the policy. Auditd and Falco are examples of auditing tools. (Falco does allow taking actions on alerts via its command execution notification channel, so it has limited enforcement capabilities, but it is not intended to be used as an enforcement tool).

Let’s start describing the tools!

<blockquote class="twitter-tweet tw-align-center" data-lang="en">
  <p dir="ltr" lang="en">
    SELinux, Seccomp, Falco, and You: A Technical Discussion <a href="https://t.co/jb04KbSWzX">https://t.co/jb04KbSWzX</a>
  </p>— Sysdig (@sysdig) 
  
  <a href="https://twitter.com/sysdig/status/807292738576072704">December 9, 2016</a>
</blockquote>

<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script> 
## Basic Sandboxing: seccomp {#seccomp}

The goal of sandboxing is to reduce the potential attack surface by reducing the actions that a process can perform. <a href="https://en.wikipedia.org/wiki/Seccomp" target="_blank" rel="noopener">Seccomp</a> is an example. Seccomp is a mechanism in the Linux kernel that allows a process to make a one-way transition to a restricted state where it can only perform a limited set of system calls. If a process attempts any other system calls, it is killed via a `SIGKILL` signal. In its most restrictive mode, seccomp prevents all system calls other than `read()`, `write()`, `_exit()`, and `sigreturn()`. This would allow a program to initialize and then drop into a restricted mode where it could only read from/write to already-opened files.

Here’s an example of using `seccomp()` in strict mode:

<script src="https://gist.github.com/3e29df625052616fffcd667ff59bf32a.js"></script> <noscript>
  <pre><code>
File: seccomp_strict.c
----------------------

#include &lt;fcntl .h>
#include &lt;stdio .h>
#include &lt;unistd .h>
#include &lt;string .h>
#include &lt;linux /seccomp.h>
#include &lt;sys /prctl.h>


int main(int argc, char **argv)
{
        int output = open("output.txt", O_WRONLY);
        const char *val = "test";

        printf("Calling prctl() to set seccomp strict mode...\n");
        prctl(PR_SET_SECCOMP, SECCOMP_MODE_STRICT);

        printf("Writing to an already open file...\n");
        write(output, val, strlen(val)+1);

        printf("Trying to open file for reading...\n");
        int input = open("output.txt", O_RDONLY);

        printf("You will not see this message--the process will be killed first\n");
}
&lt;/sys>&lt;/linux>&lt;/string>&lt;/unistd>&lt;/stdio>&lt;/fcntl></code></pre>
</noscript>

    $ ./seccomp_strict
    Calling prctl() to set seccomp strict mode...
    Writing to an already open file…
    Trying to open file for reading...
    Killed
    

You can see that after the program enables strict seccomp mode, it can write to `stdout`, which was already open, but an attempt to open a second file results in the process being killed.

## Sandboxing with Policies: seccomp-bpf {#seccomp-bpf}

Although restrictive and undoubtedly very secure, seccomp's strict mode is...strict. You can't do much other than read/write to already open files. Network activity, starting threads, even getting the current time via `gettimeofday()`, etc. are all blocked. What if you want to combine application sandboxing with flexible policies or per-application profiles to allow for a richer (but still limited) set of actions? For example, instead of preventing all file opens, you wanted to allow them only for specific files?

<a href="https://www.kernel.org/doc/Documentation/prctl/seccomp_filter.txt" target="_blank" rel="noopener">seccomp-bpf</a> is an extension to seccomp that allows specifying a filter that is applied to every system call. The filter is written using BPF, which had its origins in <a href="http://www.tcpdump.org/" target="_blank" rel="noopener">tcpdump</a>, but has become essentially a <a href="https://www.phoronix.com/scan.php?page=news_item&px=BPF-Understanding-Kernel-VM" target="_blank" rel="noopener">virtual machine</a> implementation in the Linux kernel. The BPF program loaded into the kernel starts with a system call + arguments and results in a filtering decision. Based on the results of the filter, the system call can be allowed, blocked, or the process can be killed.

Here’s an example program using seccomp with a policy that adds `open()` to the set of allowed system calls. This example is heavily inspired by, and uses the `seccomp-bpf.h` header file from, this very useful <a href="https://outflux.net/teach-seccomp/" target="_blank" rel="noopener">seccomp tutorial page</a>.

<script src="https://gist.github.com/1bc06c52abb7b6b4feef79d7bfff5815.js"></script> <noscript>
  <pre><code>
File: seccomp_policy.c
----------------------

#include &lt;fcntl .h>
#include &lt;stdio .h>
#include &lt;string .h>
#include &lt;unistd .h>
#include &lt;assert .h>
#include &lt;linux /seccomp.h>
#include &lt;sys /prctl.h>
#include "seccomp-bpf.h"

void install_syscall_filter()
{
        struct sock_filter filter[] = {
                /* Validate architecture. */
                VALIDATE_ARCHITECTURE,
                /* Grab the system call number. */
                EXAMINE_SYSCALL,
                /* List allowed syscalls. We add open() to the set of
                   allowed syscalls by the strict policy, but not
                   close(). */
                ALLOW_SYSCALL(rt_sigreturn),
#ifdef __NR_sigreturn
                ALLOW_SYSCALL(sigreturn),
#endif
                ALLOW_SYSCALL(exit_group),
                ALLOW_SYSCALL(exit),
                ALLOW_SYSCALL(read),
                ALLOW_SYSCALL(write),
                ALLOW_SYSCALL(open),
                KILL_PROCESS,
        };
        struct sock_fprog prog = {
                .len = (unsigned short)(sizeof(filter)/sizeof(filter[0])),
                .filter = filter,
        };

        assert(prctl(PR_SET_NO_NEW_PRIVS, 1, 0, 0, 0) == 0);

        assert(prctl(PR_SET_SECCOMP, SECCOMP_MODE_FILTER, &prog) == 0);
}

int main(int argc, char **argv)
{
        int output = open("output.txt", O_WRONLY);
        const char *val = "test";

        printf("Calling prctl() to set seccomp with filter...\n");

        install_syscall_filter();

        printf("Writing to an already open file...\n");
        write(output, val, strlen(val)+1);

        printf("Trying to open file for reading...\n");
        int input = open("output.txt", O_RDONLY);

        printf("Note that open() worked. However, close() will not\n");
        close(input);

        printf("You will not see this message--the process will be killed first\n");
}

&lt;/sys>&lt;/linux>&lt;/assert>&lt;/unistd>&lt;/string>&lt;/stdio>&lt;/fcntl></code></pre>
</noscript>

    $ ./seccomp_policy
    Calling prctl() to set seccomp with filter...
    Writing to an already open file...
    Trying to open file for reading...
    Note that open() worked. However, close() will not
    Bad system call
    

In strict mode, seccomp kills a process when it violates the policy. However, seccomp-bpf allows a number of actions to take based on the results of running the policy:

*   Killing the process 
*   Sending the process a `SIGSYS` signal 
*   Failing the system call and returning a (filter-provided) `errno` value 
*   Notifying an attached process tracer (see `ptrace()`), if one is attached. In turn, the process tracer can skip or even change the system call. 
*   Allowing the system call 

Probably the most widespread use of seccomp-bpf is by docker to isolate containerized applications. Docker launches processes with a <a href="https://docs.docker.com/engine/security/seccomp" target="_blank" rel="noopener">seccomp profile</a> that disables 44 system calls, preventing their use. Examples of disabled system calls are `mount` (mounting filesystems), `reboot` (reboot the host), and `setns` (change namespaces to try to escape the container).

## Mandatory Access Control Systems: AppArmor and SELinux {#mac}

In its most comprehensive form, if you add policies to sandboxing the result is <a href="https://en.wikipedia.org/wiki/Mandatory_access_control" target="_blank" rel="noopener">Mandatory Access Control</a> systems like <a href="https://wiki.ubuntu.com/AppArmor" target="_blank" rel="noopener">AppArmor</a> and <a href="https://selinuxproject.org/page/Main_Page" target="_blank" rel="noopener">SELinux</a>. These strive for system-wide enforcement of policies that control the actions and resources that each program on a system can perform. Activities outside the profile can result in logged warnings, failing the system call, or killing the process.

Here’s an example of an AppArmor profile, taken from <a href="https://wiki.ubuntu.com/AppArmor#Example_profile" target="_blank" rel="noopener">the Ubuntu AppArmor wiki</a>:

<script src="https://gist.github.com/670c94e3fa65b6a29592af838ad58f83.js"></script> <noscript>
  <pre><code>
File: usr.sbin.tcpdump.apparmor
-------------------------------

# From /etc/apparmor.d/usr.sbin.tcpdump on Ubuntu 9.04 and https://wiki.ubuntu.com/AppArmor#Example_profile

#include &lt;tunables /global>

/usr/sbin/tcpdump {
  #include &lt;abstractions /base>
  #include &lt;/abstractions>&lt;abstractions /nameservice>
  #include &lt;/abstractions>&lt;abstractions /user-tmp>

  capability net_raw,
  capability setuid,
  capability setgid,
  capability dac_override,
  network raw,
  network packet,

  # for -D
  capability sys_module,
  @{PROC}/bus/usb/ r,
  @{PROC}/bus/usb/** r,

  # for -F and -w
  audit deny @{HOME}/.* mrwkl,
  audit deny @{HOME}/.*/ rw,
  audit deny @{HOME}/.*/** mrwkl,
  audit deny @{HOME}/bin/ rw,
  audit deny @{HOME}/bin/** mrwkl,
  @{HOME}/ r,
  @{HOME}/** rw,

  /usr/sbin/tcpdump r,
}
&lt;/abstractions>&lt;/tunables></code></pre>
</noscript>

The profile begins with a program `/usr/bin/tcpdump`, and then declares what the program can do, including:

*   Permitted linux <a href="http://man7.org/linux/man-pages/man7/capabilities.7.html" target="_blank" rel="noopener">capabilities</a> : `net_raw`, `setuid`, `setgid`, `dac_override`
*   Permitted <a href="http://wiki.apparmor.net/index.php/AppArmor_Core_Policy_Reference#Network_rules" target="_blank" rel="noopener">network operations</a>: `raw`, `packet`
*   Allowed files: `/proc/bus/usb` and its children, files below `$HOME`, and `/usr/bin/tcpdump` itself.
*   Disallowed files: any dot-files or dot-directories below `$HOME` and anything below `$HOME/bin`. `audit deny` also indicates that attempts to access these files should be logged.

Unlike AppArmor, where profiles are oriented around processes, SELinux policies are much more complex, and apply separately to actors, actions, and targets, with a whole middleware of types defining the policies for each in different places. I won’t try to explain all these SELinux concepts here -- <a href="https://wiki.gentoo.org/wiki/SELinux/Tutorials" target="_blank" rel="noopener">this tutorial</a> is a longer but more thorough introduction. However, let’s briefly look at the set of policies for `/usr/sbin/tcpdump` to see how it’s allowed to access raw sockets (the `net_raw` capability above).

Let’s start by using `ls -Z` to show the security context information for `tcpdump`:

    $ ls -Z /usr/sbin/tcpdump
    -rwxr-xr-x. root root system_u:object_r:netutils_exec_t:s0 /usr/sbin/tcpdump
    

The security context information includes a user (`system_u`), a role (`object_r`), a type (`netutils_exec_t`) and a level (`s0`). In this case, it means that when someone runs `tcpdump` the running process changes to a security context with the `netutils_t` type. We can see that by using the SELinux search tool `sesearch`:

    $ sesearch -t netutils_exec_t -c file -p entrypoint -Ad
    Found 1 semantic av rules:
       allow netutils_t netutils_exec_t : file { ioctl read getattr lock execute execute_no_trans entrypoint open } ;
    

In plain english, that command line is “show me the domain transitions involving `netutils_exec_t` as an entrypoint, when related to files”.

If we want to see what linux capabilities programs with the context `netutils_t` have, we can use `sesearch` again:

    # sesearch -t netutils_t -c capability -Ad
    Found 1 semantic av rules:
       allow netutils_t netutils_t : capability { chown dac_read_search setgid setuid net_admin net_raw sys_chroot } ;
    

In plain english, that command line is “show me the linux capabilities associated with the `netutils_t` type.

The distinction between AppArmor, SELinux, and Seccomp can be fuzzy at times. They all rely on the same mechanisms--kernel-level interception/filtering of system calls, driven by per-process policies. However, there are some distinctions.

The policy languages used by AppArmor and SELinux differ from each other with respect to ease of use and specific terminology, but are generally richer and more complex than seccomp. Both allow for defining actors (generally processes), actions (reading files, network operations), and targets (files, IPs, protocols, etc.), rather than a simple list of system calls and arguments.

Additionally, seccomp is voluntary. Processes have to willingly drop into a restricted state by calling `prctl(PR_SET_SECCOMP, …)`, in Mandatory Access Control systems the policy is defined and loaded before a process is run.

## System Auditing: Auditd {#auditd}

Auditd is an access monitoring system, and is actually used as the logger for SELinux. Auditd's configuration consists of a series of rules that add monitoring points either on filesystem paths or system calls. When the relevant file is accessed and/or the system call is called, details on the action are logged.

Here’s an example set of Auditd rules:

<script src="https://gist.github.com/06c010eb23aed2ebf970d8ca758e710f.js"></script> <noscript>
  <pre><code>
File: sample_auditd.rules
-------------------------

# Alert whenever anyone performs the mount() system call.
-a always,exit -S mount

# Alert whenever anyone performs an unlink() for a file below /usr/bin
-a always,exit -S unlink -S unlinkat -F dir=/usr/bin -F success=1

# Watch all activity related to /etc/shadow
# -k puts this rule and the following rule in a group
-w /etc/shadow -p wa -k passwd_mgmt

# Watch any invocation of /usr/bin/passwd
-w /usr/bin/passwd -p x -k passwd_mgmt

# Lock this configuration so it can't be changed.
-e 2
</code></pre>
</noscript>

The policy watches for any `mount()` syscalls, file removals below `/usr/bin`, and any attempt to modify `/etc/shadow` or run `passwd`.

If we violate the policy via `touch /usr/bin/foo && rm /usr/bin/foo`, here’s the log messages that result:

    type=SYSCALL msg=audit(1479512983.650:523): arch=c000003e syscall=263 success=yes exit=0 a0=ffffffffffffff9c a1=1c180c0 a2=0 a3=7ffd261e6350 items=2 ppid=3471 pid=3860 auid=1000 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=pts1 ses=2 comm="rm" exe="/usr/bin/rm" subj=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023 key=(null)
    type=CWD msg=audit(1479512983.650:523):  cwd="/home/mstemm"
    type=PATH msg=audit(1479512983.650:523): item=0 name="/usr/bin/" inode=33554571 dev=fd:00 mode=040555 ouid=0 ogid=0 rdev=00:00 obj=system_u:object_r:bin_t:s0 objtype=PARENT
    type=PATH msg=audit(1479512983.650:523): item=1 name="/usr/bin/foo" inode=34213876 dev=fd:00 mode=0100644 ouid=0 ogid=0 rdev=00:00 obj=unconfined_u:object_r:bin_t:s0 objtype=DELETE
    

You can see that auditd detected the unlink of `/usr/bin/foo` and provided details on the activity.

## Behavioral Monitoring: Sysdig Falco {#falco}

Hopefully you already have an idea how Sysdig Falco works, but just as a refresher, Falco is a behavioral activity monitor designed to detect anomalous activity in applications. Falco's policies are a collection of rules that act on a stream of system calls from the kernel. Rules use <a href="https://github.com/draios/sysdig/wiki/Sysdig-User-Guide#filtering" target="_blank" rel="noopener">Sysdig filtering expressions</a> to identify suspicious activity and send notifications to either files, syslog, and/or programs.

Falco’s capabilities go beyond monitoring of individual system calls. Because Falco is built on top of sysdig’s event processing libraries, system calls are turned into *events* that include the context in which a system call was performed, including:

*   the process name performing the system call
*   the process's parents, grandparents, etc.
*   the remote IP address to which the process is communicating
*   the directory of the file being read/written
*   the current memory usage of the process
*   etc.

In addition, Falco 0.4.0 can communicate with k8s/mesos servers and define policies using the terminology of containers and orchestration frameworks like: container ids, image names, Kubernetes namespaces, services, deployments or Mesos frameworks, etc.

Here’s an example Falco rule:

<script src="https://gist.github.com/abd3d875fe94314496f220a38e02983f.js"></script> <noscript>
  <pre><code>
File: falco_sample_rules.yaml
-----------------------------

- macro: bin_dir
  condition: fd.directory in (/bin, /sbin, /usr/bin, /usr/sbin)
  
- macro: open_write
  condition: (evt.type=open or evt.type=openat) and evt.is_open_write=true and fd.typechar='f'
    
- macro: package_mgmt_binaries
  items: [dpkg, dpkg-preconfigu, rpm, rpmkey, yum, frontend]

- rule: Write below binary dir
  desc: an attempt to write to any file below a set of binary directories
  condition: bin_dir and evt.dir = &lt; and open_write and not proc.name in (package_mgmt_binaries)
  output: "File below a known binary directory opened for writing (user=%user.name command=%proc.cmdline file=%fd.name)"
  priority: WARNING
  
</code>&lt;/code></pre>
</noscript>

The heart of each rule is its condition field, which is a filter applied to each system call. When an event matches the condition expression, the output field is used to format a notification message using a mix of plain text and information from the event. Note that the rules file also contains *macros* (filtering expression snippets) and *lists* (lists of processes, files, etc.) that allow for easy code re-use.

In this case, the rule monitors file opens to identify attempts to open a file below one of a number of binary directories. If you attempted to write to a file, for example:

    # touch /bin/hack
    

Here’s the message that results:

    09:44:39.349828144: Warning File below a known binary directory opened for writing (user=root command=touch /bin/hack file=/bin/hack)
    

Here’s an example rule that highlights the ability to use higher level context like container information to identify suspicious behavior:

<script src="https://gist.github.com/a95966413162e990bfee8a9fdea86067.js"></script> <noscript>
  <pre><code>
File: Falco 0.4.0 Sample Rules
------------------------------

- rule: File Open by Privileged Container
  desc: Any open by a privileged container. Exceptions are made for known trusted images.
  condition: (open_read or open_write) and container and container.privileged=true and not trusted_containers
  output: File opened for read/write by non-privileged container (user=%user.name command=%proc.cmdline %container.info file=%fd.name)
  priority: WARNING

- macro: sensitive_mount
  condition: (container.mount.dest[/proc*] != "N/A")

- rule: Sensitive Mount by Container
  desc: Any open by a container that has a mount from a sensitive host directory (i.e. /proc). Exceptions are made for known trusted images.
  condition: (open_read or open_write) and container and sensitive_mount and not trusted_containers
  output: File opened for read/write by container mounting sensitive directory (user=%user.name command=%proc.cmdline %container.info file=%fd.name)
  priority: WARNING
</code></pre>
</noscript>

These rules identify file opens where the process opening the file is in a privileged container or is in a container that has mounted one of a number of sensitive filesystems like `/proc`. Although the system call is the event that triggers the suspicious behavior, it’s the *context* (in this case, the properties of the container in which the open occurred) that makes the behavior suspicious.

## How Sysdig Falco Fits In {#howfalcofitsin}

Now that we've reviewed other security tools in the landscape, let's take a closer look at Falco and compare its approach to these other tools.

All of these tools use the same general approach--define a policy that names the actions that programs can/can not perform, and then apply that policy to the stream of system calls on a machine. One difference between Falco and other tools is that Falco runs in user space, using a kernel module to obtain system calls, while the other tools perform system call filtering/monitoring at the kernel level. This makes Falco a somewhat softer target. If you can kill/suspend/starve the Falco process you can disable detection. Replacing a loaded set of policies or BPF program in the kernel is probably more difficult.

On the other hand, because Falco runs as a userspace process it can have a much richer set of information powering its policies, as we showed previously. Those types of policies are more difficult to implement at the kernel level. Seccomp-bpf allows executing virtual "programs" using BPF, but its inputs are still a system call + arguments. AppArmor and SELinux can aggregate system call information into higher-level state, but can not augment that state from external sources like k8s/mesos API servers. Auditd's filtering is limited to actions on filesystem paths and system calls + arguments.

Another distinction is that the policy language for SELinux (and to a lesser extent, AppArmor) can have a steep learning curve. It's certainly powerful, but it can take a lot of steps to create the necessary users, roles, subjects, and targets and tie them together into a policy. For example, here’s how you would implement the SELinux example (ensuring that only tcpdump has access to raw network sockets) as a Falco rule:

<script src="https://gist.github.com/2969b75f7fb7552adc9d4c610a6e369a.js"></script> <noscript>
  <pre><code>
File: packet_sockets_using_falco.yaml
-------------------------------------

- rule: raw_network_socket
  desc: an attempt to open a raw network socket by an unexpected program
  condition: evt.type=socket and evt.dir=> and evt.arg.domain=AF_PACKET and not proc.name=tcpdump
  output: Raw network socket opened by unexpected program (user=%user.name command=%proc.cmdline domain=%evt.arg.domain)
  priority: WARNING
</code></pre>
</noscript>

Instead of a scattered list of capabilities spread across multiple files, you can tie the process, action, and arguments together into a single rule that identifies processes creating raw packet sockets that aren’t in a list of allowed processes.

Overall, we think the filtering language used by Sysdig which is at the heart of Falco rules is a bit easier-to-use than other tools. But we're obviously biased :)

## Conclusion {#conclusion}

Hopefully this has helped show how Falco fits into the overall security landscape. Now go <a href="https://www.sysdig.org/falco/" target="_blank" rel="noopener">download</a> it and try it out for yourself! You can also check out our <a href="https://fosdem.org/2017/schedule/event/container_spawned_shell/" target="_blank" rel="noopener">presentation on Sysdig Falco at FOSDEM 2017: WTF my container just spawned a shell!</a> ;-)

<iframe width="595" height="485" style="border: 1px solid #CCC; border-width: 1px; margin-bottom: 5px; max-width: 100%;" src="//www.slideshare.net/slideshow/embed_code/key/ewPwqg39rC6NaY" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" allowfullscreen="allowfullscreen"> </iframe>

<div style="margin-bottom: 5px;">
  <strong> <a href="//www.slideshare.net/Sysdig/wtf-my-container-just-spawned-a-shell" title="WTF my container just spawned a shell!" target="_blank" rel="noopener">WTF my container just spawned a shell!</a> </strong> from <strong><a target="_blank" href="https://www.slideshare.net/Sysdig" rel="noopener">Sysdig </a></strong>
</div>

 [1]: #seccomp
 [2]: #seccomp-bpf
 [3]: #mac
 [4]: #auditd
 [5]: #falco