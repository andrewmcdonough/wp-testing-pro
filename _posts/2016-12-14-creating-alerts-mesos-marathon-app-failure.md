---
ID: 3434
post_title: >
  Creating alerts for mesos and marathon
  app failure
author: Kamol Mavlonov
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/creating-alerts-mesos-marathon-app-failure/
published: true
post_date: 2016-12-14 12:01:11
---
This blog was initially posted by an awesome member of the sysdig community, Kamol Mavlonov on <http://blog.microservices.today/>. If you're new to alerting in sysdig cloud this [how to video][1] is a great place to start! 
### Creating alerts for marathon failure {#creatingalertsformarathonfailure}

1.  Under Explore tab select Server -> Overview.
2.  Choose Group by `mesos.framework.name`.
3.  Click on the bell button next to marathon framework ( marathon [<ip>:8080] ) from the list. A new alert popup will appear.</ip>
4.  Under `Set the condition` choose type as `manual`
5.  For `Alert when` option Choose `Entity is down`.
6.  Leave `Where` option unchecked.
7.  Choose the minimum monitoring value as `1 min`.
8.  Specify the Name, Description and Severity of the alert.
9.  Enable the notification channel.
10. Enable automatic sysdig capture if necessary.
11. Click Create button.

![Creating alerts for marathon failure][2]

While grouping with `mesos.framework.name` on sysdig, it lists marathon from work running on one of the masters ( marathon [<ip>:8080] ). But the ip showing in the list might change when marathon is restarted. In that case we need to create alert for new listing name also. For solving this issue, we need to create alert for each of the master nodes. In order to do that:</ip>

*   Go to Alerts page and select the alert we have created from above steps. Click copy for creating a similar alert. 
*   Modify the master ip from the scope manually. 
*   Specify the ip of another master and create a new alert. 
*   Similarly create alert for each of the master nodes.

![Set the scope of nodes to watch][3]

### Creating alerts for marathon app failure {#creatingalertsformarathonappfailure}

1.  Under Explore tab select Server -> Overview.

2.  Choose Group by `agent.tag.dcosName`.

3.  Click on the bell button next to your dcos name 
    
    a. A new alert popup will appear.

4.  Under `Set the condition` choose type as `manual`

5.  For `Alert when` option Choose `Entity is down`.

6.  For `Segmented by` option choose second radio button and select `Any of` from dropdown. Select the filter metrics `marathon.app.name` from the dropdown.

7.  Leave `Where` option unchecked.

8.  Choose the minimum monitoring value as `1 min`.

9.  Specify the Name, Description and Severity of the alert.

10. Enable the notification channel.

11. Enable automatic sysdig capture if necessary.

12. Click Create button.

![Creating alerts for marathon app failure][4]

 [1]: https://www.youtube.com/watch?v=rsaYYJOnK6s&index=6&list=PLrUjPk-W0lae7KuCFvmdbWj9Powm7Ryu0
 [2]: /wp-content/uploads/2016/12/Creating-alerts-for-marathon-failure.png
 [3]: /wp-content/uploads/2016/12/Creating-alerts-for-marathon-failure2.png
 [4]: /wp-content/uploads/2016/12/Creating-alerts-for-marathon-app-failure.png