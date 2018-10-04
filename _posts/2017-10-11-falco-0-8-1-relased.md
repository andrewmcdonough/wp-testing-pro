---
ID: 4655
post_title: Falco 0.8.1 Released
author: Mark Stemm
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/falco-0-8-1-relased/
published: true
post_date: 2017-10-11 20:22:13
---
We just released Falco 0.8.1. This has a great list of new features and rule improvements. 

## Rule Improvements



The ruleset has undergone a major set of updates to reduce false positives and improve coverage. Nearly every rule has been modified. These improvements were based on beta testing with existing Sysdig customers and represent thousands of hours of real-world usage with the ruleset. Thanks so much to all of the beta customers for the valuable feedback and testing! 

## Local vs Default Rules



Starting with Falco 0.8.1, falco officially supports the notion of a *default* rules file and a *local* rules file. This has previously been supported by running falco with multiple -r arguments, but in 0.8.1 we're formalizing this notion to make it easier to customize falco's behavior but still retain access to rule changes as a part of software upgrades. 

The intent is that the default rules file remains unmodified and is replaced with every new release, while the local rules file contains extensions and modifications to the default rules file. The default rules file has been moved from <tt>/etc/falco_rules.yaml</tt> to <tt>/etc/falco/falco_rules.yaml</tt>. The local rules file is now at <tt>/etc/falco/falco_rules.local.yaml</tt>. We also moved the falco config file from <tt>/etc/falco.yaml</tt> to <tt>/etc/falco/falco.yaml</tt> for consistency. 

The RPM/Debian Falco packages now flag all 3 config files as config files, so they are not overwritten/removed on upgrade if they have been locally modified. 

## Extendable Rules, Macros, and Lists



To further support the notion of extensibility, we made it easier to extend lists/macros/rules in a local rules file by adding an *append* attribute. If true, the contents of the later list/macro/rule are added to an existing list/macro/rule with the same name. Here's an example: 

**<tt>/etc/falco/falco_rules.yaml</tt>** 

    - list: my_programs
      items: [ls, cat, pwd]
    
    - macro: access_file
      condition: evt.type=open
    
    - rule: program_accesses_file
      desc: track whenever a set of programs opens a file
      condition: proc.name in (cat, ls) and evt.type=open
      output: a tracked program opened a file (user=%user.name command=%proc.cmdline file=%fd.name)
      priority: INFO



**<tt>/etc/falco/falco_rules.local.yaml</tt>** 

    - list: my_programs
      append: true
      items: [cp]
    
    - macro: access_file
      append: true
      condition: or evt.type=openat
    
    - rule: program_accesses_file
      append: true
      condition: and not user.name=root



The list my_programs would contain the programs <tt>[ls, cat, pwd, cp]</tt>. The condition for the access_file macro would be <tt>evt.type=open or evt.type=openat</tt>. The condition for the program_accesses_file rule would be <tt>proc.name in (cat, ls) and evt.type=open and not user.name=root</tt>. 

These changes should make it easier to customize a list/macro/rule without having to copy the entire item and override it. 

## Making it Easier to Send Alerts



We've also made it easier to send alerts to external programs. When using the program output channel, if you set the attribute *keep_alive* to true the program is spawned once rather than once for every alert. This allows use of long-lived programs (e.g. netcat) to stream alerts over a network connection. Here's an example: 

    # Whether to output events in json or text
    json_output: true
    …
    program_output:
      enabled: true
      keep_alive: true
      program: "nc host.example.com 1234"



Additionally, when using json output the individual templated fields of the output message are sent in the object along with the time, full output string, etc. This makes it easier for downstream programs to parse individual fields of an alert. Here's an example: 

    {
       "output" : "16:31:56.746609046: Error File below a known binary directory opened for writing (user=root command=touch /bin/hack file=/bin/hack)"
       "priority" : "Error",
       "rule" : "Write below binary dir",
       "time" : "2017-10-09T23:31:56.746609046Z",
       "output_fields" : {
          "user.name" : "root",
          "evt.time" : 1507591916746609046,
          "fd.name" : "/bin/hack",
          "proc.cmdline" : "touch /bin/hack"
       }
    }



## Other Changes



Some other improvements include: 

*   The ability to send unbuffered data to output channels via the <tt>--unbuffered</tt> option. 
*   The ability to validate a single rules file and exit, via the <tt>-V</tt> option. Limiting the rules that run by severity. For example using <tt>-o priority=info</tt> would skip all debug priority rules. 
*   Improve parsing of strings that contain trailing whitespace or dot characters. 



## Learn More {#learnmore}



For the full set of changes in this release, you can always look at the [changelog at github][1].



The release is available via the usual channels–rpm/debian packages, [docker hub][2] and [github][3].



Let us know if you have any issues, and enjoy!

 [1]: https://github.com/draios/falco/blob/dev/CHANGELOG.md
 [2]: https://hub.docker.com/r/sysdig/falco
 [3]: https://github.com/draios/falco