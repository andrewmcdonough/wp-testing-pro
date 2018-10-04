---
ID: 3500
post_title: 'Creating utilization alerts &#038; dashboards for Mesos &#038; DCOS'
author: Kamol Mavlonov
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/creating-utilization-alerts-mesos-dcos/
published: true
post_date: 2017-01-03 09:42:47
---
This blog was initially posted by an awesome member of the sysdig community, Kamol Mavlonov on <http://blog.microservices.today/>. He covers how to get up and running with Dashboards and Alerts to monitor the CPU, Memory, and Disk utilization in your mesos environments.

### Creating a Disk Utilization Dashboard {#creatingadashboardtab}

1.  Under Explore tab select Server -> Overview.
2.  Choose Group by host (host.mac).
3.  On Table columns configuration (gear icon) Select the following fields

*   `fs.used.percent` - FS Usage %
*   `fs.root.used.percent` - FS Root Usage %
*   `fs.largest.used.percent` - FS Largest Usage %
*   `fs.bytes.total` - FS Size
*   `fs.bytes.free` - FS Free Space
*   `fs.bytes.used` - Disk Used Bytes

1.  Change the color coding by clicking on the gear icon of each of the columns. (default is yellow after 50% and red after 80%)
2.  Pin the tab to the dashboard.

![creating a dashboard tab][1]

### Creating an alert for 60% disk utilization {#creatinganalertfor60diskutilization}

The following steps will help you to create an alert when the root directory (/) exceeds 60% of its usage:

1.  Under alert tab click add alert button
2.  Select the scope as `agent.tag.dcosName` or `region`. `agent.tag.dcosName` Is the tag we have added for the dcos cluster. `region` specify the aws region name. Assign the value for scope from the dropdown accordingly which one you selected as the scope.
3.  Under `Set the condition` choose type as `manual`
4.  For `Alert when` option Choose `fs.used.percent` as the metric > 60% as the threshold value.
5.  For `Segment by` choose second option, select `Any of` and `fs.mountDir` metric. ![creating an alert for disk utilization][2]
6.  Check the `Where` option and select `fs.mountDir` from the dropdown. Assign the value `/` ![creating an alert for disk][3]
7.  Choose the minimum monitor value as `1 min`.
8.  Specify the Name, Description and Severity of the alert.
9.  Enable the notification channel.
10. Enable automatic sysdig capture if necessary.
11. Click Create button.

### Creating an alert for 95% CPU utilization {#creatinganalertfor95cpuutilization}

The following steps will help you to create an alert when a node exceeds 90% of its memory utilization.

1.  Under alert tab click add alert button
2.  Select the scope as `agent.tag.dcosName` or `region`. `agent.tag.dcosName` Is the tag we have added for the dcos cluster. `region` specify the aws region name. Assign the value for scope from the drop down accordingly which one you selected as the scope.
3.  Under `Set the condition` choose type as `manual`
4.  For `Alert when` option Choose `memory.used.percent` as the metric > 95% as the threshold value.
5.  For `Segment by` choose second option, select `Any of` and `host.hostName` metric.
6.  Leave the `Where` option unchecked.
7.  Choose the minimum monitor value as `5 min`.
8.  Specify the Name, Description and Severity of the alert.
9.  Enable the notification channel.
10. Enable automatic sysdig capture if necessary.
11. Click Create button. ![creating an alert for CPU utilization][4]

### Creating an alert for 90% CPU utilization {#creatinganalertfor90cpuutilization}

The following steps will help you to create an alert when a node exceeds 90% of its cpu utilization:

1.  Under alert tab click add alert button.
2.  Select the scope as `agent.tag.dcosName` or `region`. `agent.tag.dcosName` Is the tag we have added for the dcos cluster. `region` specify the aws region name. Assign the value for scope from the drop down accordingly which one you selected as the scope.
3.  Under `Set the condition` choose type as `manual`
4.  For `Alert when` option Choose `cpu.used.percent` as the metric > 90% as the threshold value.
5.  For `Segment by` choose second option, select `Any of` and `host.hostName` metric.
6.  Leave the `Where` option unchecked.
7.  Choose the minimum monitor value as `5 min`.
8.  Specify the Name, Description and Severity of the alert.
9.  Enable the notification channel.
10. Enable automatic sysdig capture if necessary.
11. Click Create button.

![creating an alert for 90% CPU utilization][5]

 [1]: /wp-content/uploads/2017/01/Disk-utilization-monitoring-using-Sysdig-dashboard-tab.png
 [2]: /wp-content/uploads/2017/01/Creating-alert-for-disk-utilization.png
 [3]: /wp-content/uploads/2017/01/Creating-alert-for-disk-utilization2.png
 [4]: /wp-content/uploads/2017/01/Memory-utilization-monitoring-using-Sysdig.png
 [5]: /wp-content/uploads/2017/01/Creating-an-alert-for-90-CPU-utilization.png