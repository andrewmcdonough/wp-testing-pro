---
ID: 6629
post_title: 'Sending Kubernetes &#038; Docker events to Elasticsearch and Splunk using Sysdig'
author: Mateo Burillo
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/kubernetes-docker-elasticsearch-splunk/
published: true
post_date: 2018-03-06 10:34:15
---
In this article we are going to see how to aggregate Kubernetes / Docker events and alerts into a centralized logs system like <a href="https://www.elastic.co/" target="_blank">Elasticsearch</a> and <a href="https://www.splunk.com/" target="_blank">Splunk</a>.

Logging engines are a great companion of Kubernetes monitoring like Sysdig Monitor. Typically responding an incident begins looking at the relevant metrics and then finding out if there are any related log entries.

In the context of security, bringing together events from different sources can shed some light on the reach of the breach. Sysdig Secure can emit secure policy violation events, but also block the attacks and enable post-mortem analysis and forensics.

## Comparing events notification channels {#comparingeventsnotificationchannels}

Both <a href="https://sysdigrp2rs.wpengine.com/product/monitor/" target="_blank">Sysdig Monitor</a> and <a href="https://sysdigrp2rs.wpengine.com/product/secure/" target="_blank">Sysdig Secure</a> provide powerful semantics and notification channels to define the events and alerts that you want to monitor.

If you access the *Notifications* section <a href="https://secure.sysdigrp2rs.wpengine.com/#/settings/notifications" target="_blank">of your profile on Sysdig Secure</a> or <a href="https://app.sysdigcloud.com/#/settings/notifications" target="_blank">on Sysdig Monitor</a>, you will find the list of integrated notification channels. For any alert on metric threshold, event or security incident you can configure one or more of these notification channels:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/sysdig_notification_channels.png" alt="Sysdig Notification Channels" width="943" height="482" class="alignleft size-full wp-image-6639" />][1] 

*   **Email:** Old school notifications. Goes directly to your inbox and doesn't need any other additional requirements. Email has its own limitations (no delivery guarantees, no acknowledgment, or integration with other software for escalation channels, rotation, etc). Still very used due to the low entry barrier.
*   **<a href="https://www.pagerduty.com/" target="_blank">PagerDuty</a>:** SaaS commercial product for incident response platform specifically tailored for IT and support teams.
*   **<a href="https://slack.com/" target="_blank">Slack</a>:** Having informal notification channels in your enterprise messaging platform is increasingly popular, it encourages agile issue discussion and team awareness.
*   **Amazon SNS:** Cloud-native Amazon Simple Notification Service (SNS), a <a href="https://aws.amazon.com/pub-sub-messaging/" target="_blank">pub/sub messaging</a> and mobile notifications service, typically used when you build your own events / alerts management service.
*   **<a href="https://victorops.com/" target="_blank">VictorOps</a>:** SaaS commercial product for DevOps oriented incident management solution. Focused on "on-call" IT engineers and best practices to minimize downtime.
*   **<a href="https://www.opsgenie.com/" target="_blank">OpsGenie</a>:** Another commercial product for alerting, on-call management and incident response orchestration solution.

And, if you need to integrate with anything else… then you have the ubiquitous WebHook: a user-defined HTTP callback.

[tweet_box design="default" float="none"]Sending #Kubernetes & #Docker events to #Elasticsearch and #Splunk using @Sysdig[/tweet_box] 

The IT world has been trying to standardize over a platform agnostic <a href="https://en.wikipedia.org/wiki/Remote_procedure_call" target="_blank">remote procedure call</a> protocol for a long time. Just to name a few, you have SOAP, CORBA, and lately, very often used in the Docker and Kubernetes ecosystem: <a href="https://grpc.io/" target="_blank">gRPC</a>.

The WebHook mechanism is one of the most popular and common on the web, due to its inherent simplicity and predictability:

*   It builds over well known languages and protocols HTTP and <a href="https://www.json.org/" target="_blank">JSON</a>.
*   Can be exposed and processed using commonplace web servers.
*   Data manipulation and retrieval verbs usually follow the <a href="https://en.wikipedia.org/wiki/Representational_state_transfer" target="_blank">REST</a> design patterns.

Let's create a WebHook callback to send Sysdig events to Elasticsearch and Splunk as an example!

## Configuring Elasticsearch and Splunk WebHook integration {#configuringelasticsearchandsplunkwebhookintegration}

These two platforms accept regular HTTP calls with JSON body content as one of their data input mechanism, making forwarding from Sysdig easy enough.

We can add a new WebHook notification channel on Sysdig with your Elasticsearch or Splunk URL.

*   For Elasticsearch, we follow the URL convention */index/type*. You can see in the example below that we specify the host and port and then use the index *sysdigsecure* and document type *event*. Elastic provides plenty of documentation on how to <a href="https://www.elastic.co/guide/en/elasticsearch/guide/current/index-doc.html" target="_blank">index documents</a> and how to <a href="https://www.elastic.co/guide/en/elasticsearch/guide/current/data-in-data-out.html" target="_blank">send data to the engine</a>.
    
    Give it a name and also define if you want to send over trigger off and resolved notifications.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/sysdig_elasticsearch_webhook-1024x366.png" alt="Sysdig elasticsearch webhook" width="1024" height="366" class="alignleft size-large wp-image-6638" />][2] 

*   For Splunk, we have created an HTTP event collector or <a href="http://dev.splunk.com/view/event-collector/SP-CAAAE6M" target="_blank">HEC</a>. Splunk events have their own JSON schema, we used the <a href="http://dev.splunk.com/view/event-collector/SP-CAAAE8Y#raw" target="_blank">"raw" endpoint</a> *services/collector/raw* so we can keep the original Sysdig formatting and semantics.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/sysdig_splunk_webhook-1024x353.png" alt="Sysdig splunk webhook" width="1024" height="353" class="alignleft size-large wp-image-6643" />][3] 

There are, however, two more details that you need to adjust for this to work as intended: **authentication** and **value mapping / preprocessing**.

### WebHook authentication using headers {#webhookauthenticationusingheaders}

It's highly recommended not just sending data over HTTPS but also to configure WebHook authentication <a href="https://sysdigrp2rs.wpengine.com/blog/sysdig-monitor-winter-2017-8-release/#customheadersforwebhooks" target="_blank">using custom headers</a> on your HTTP requests.

In order to define these custom headers, we will use the Sysdig API to modify the previously created notification channels:

First, get your API Token for this operation <a href="https://app.sysdigcloud.com/#/settings/user" target="_blank">on Sysdig Monitor</a> or <a href="https://secure.sysdigrp2rs.wpengine.com/#/settings/user" target="_blank">on Sysdig Secure</a>.

Second, retrieve the list of configured notification channels (BEARER-CODE is the API Token we just mentioned):

    curl -X GET \
      https://app.sysdigcloud.com/api/notificationChannels\
      -H 'authorization: Bearer BEARER-CODE-HERE'

And locate the channel you want to modify in the JSON output description

<script src="https://gist.github.com/e6729c6e2584acd00c7a4e18f3f26fbe.js"></script> <noscript>
  <pre><code>
File: sysdigevent.json
----------------------

...  
  {
      "id": 14787,
      "version": 17,
      "createdOn": 1511433414000,
      "modifiedOn": 1519827129000,
      "type": "WEBHOOK",
      "enabled": true,
      "sendTestNotification": true,
      "name": "Splunk",
      "options": {
        "notifyOnOk": true,
        "url": "http://somehost:9000/services/collector/raw",
        "notifyOnResolve": true
      }
    }
...
</code></pre>
</noscript>

OK, so for this example we want to modify the channel 14787, version 17.

Third and last, let's update with our authentication header. We will use the <a href="http://docs.splunk.com/Documentation/Splunk/7.0.1/Data/UsetheHTTPEventCollector#JSON_request_and_response" target="_blank">Splunk HEC token and auth header format</a> for this example:

    curl -X PUT \
      https://app.sysdigcloud.com/api/notificationChannels/14787 \
      -H 'Authorization: Bearer BEARER-CODE-HERE' \
      -H 'Content-Type: application/json' \
      -d '{
      "notificationChannel": {
        "id": 14787,
        "version": 17,
        "type": "WEBHOOK",
        "enabled": true,
        "name": "Splunk",
        "options": {
          "notifyOnOk": true,
          "url": "http://somehost:9000/services/collector/raw",
          "notifyOnResolve": true,
          "additionalHeaders": {
            "Authorization": "Splunk 2b8868fe198cf1203256e6af6515bfad"
          }
        }
      }
    }'

Note that I copied the field values from the original channel, adding the *additionalHeaders* dictionary.

Retrieving the available channels again you should get:

<script src="https://gist.github.com/9429f9d948082ae662751c83ee6c10f2.js"></script> <noscript>
  <pre><code>
File: sysdigevent2.json
-----------------------

  {
      "id": 14787,
      "version": 18,
      "createdOn": 1511433414000,
      "modifiedOn": 1519828577000,
      "type": "WEBHOOK",
      "enabled": true,
      "sendTestNotification": false,
      "name": "Splunk",
      "options": {
        "notifyOnOk": true,
        "url": "http://somehost:9000/services/collector/raw",
        "additionalHeaders": {
          "Authorization": "Splunk 2b8868fe198cf1203256e6af6515bfad"
        },
        "notifyOnResolve": true
      }
    },
</code></pre>
</noscript>

This update bumped the version to 18, adding the HTTP Authorization header.

## Elasticsearch and Splunk data preprocessing for Kubernetes & Docker events {#elasticsearchandsplunkdatapreprocessingforkubernetesdockerevents}

Every data analytics engine has its own data formatting preferences and conventions. You will probably need to configure some preprocessing to adjust incoming data.

Elasticsearch <a href="https://www.elastic.co/guide/en/elasticsearch/reference/1.4/mapping.html" target="_blank">mapping</a> and ingest nodes or Splunk <a href="https://docs.splunk.com/Documentation/Splunk/7.0.2/Knowledge/WhenSplunkEnterpriseaddsfields" target="_blank">field extraction</a> are huge topics, out of scope for this article, but just as an example, consider this Sysdig Secure event instance:

<script src="https://gist.github.com/605928250d165df0fa37f3181378a6b6.js"></script> <noscript>
  <pre><code>
File: sysdigsecureevent.json
----------------------------

{
	"timestamp": 1518849360000000,
	"timespan": 60000000,
	"alert": {
		"severity": 4,
		"editUrl": null,
		"scope": null,
		"name": "Policy 59: FILE POLICY: Read sensitive file untrusted",
		"description": "an attempt to read any sensitive file (e.g. files containing user/password/authentication information). Exceptions are made for known trusted programs.",
		"id": null
	},
	"event": {
		"id": null,
		"url": "https://secure.sysdigrp2rs.wpengine.com/#/events/f:1518849300,t:1518849360"
	},
	"state": "ACTIVE",
	"resolved": false,
	"entities": [{
		"entity": "",
		"metricValues": [{
			"metric": "policyEvent",
			"aggregation": "count",
			"groupAggregation": "none",
			"value": 1
		}],
		"additionalInfo": null,
		"policies": [{
			"id": 59,
			"version": 9,
			"createdOn": 1496775488000,
			"modifiedOn": 1512474141000,
			"name": "FILE POLICY: Read sensitive file untrusted",
			"description": "an attempt to read any sensitive file (e.g. files containing user/password/authentication information). Exceptions are made for known trusted programs.",
			"severity": 4,
			"enabled": true,
			"hostScope": true,
			"containerScope": true,
			"falcoConfiguration": {
				"onDefault": "DEFAULT_MATCH_EFFECT_NEXT",
				"fields": [],
				"ruleNameRegEx": "Read sensitive file untrusted"
			},
			"notificationChannelIds": [
				14872
			],
			"actions": [{
				"type": "POLICY_ACTION_CAPTURE",
				"beforeEventNs": 30000000000,
				"afterEventNs": 30000000000,
				"isLimitedToContainer": false
			}],
			"policyEventsCount": 295,
			"isBuiltin": false,
			"isManual": true
		}],
		"policyEvents": [{
			"id": "513051281863028736",
			"version": 1,
			"containerId": "57c1820a87f1",
			"severity": 4,
			"metrics": [
				"ip-10-0-8-165",
				"k8s_ftest_redis-3463099497-2xxw3_example-java-app_08285988-acff-11e7-b6b2-06fd27f1a4ca_0"
			],
			"policyId": 59,
			"actionResults": [{
				"type": "POLICY_ACTION_CAPTURE",
				"successful": true,
				"token": "e0abbbfb-ae65-4c5d-966a-78f88b0f67fb",
				"sysdigCaptureId": 432336
			}],
			"output": "Sensitive file opened for reading by non-trusted program (user=root name=ftest command=ftest -i 25200 -a exfiltration file=/etc/shadow parent=docker-containe gparent=docker-containe ggparent=dockerd gggparent=systemd)",
			"ruleType": "RULE_TYPE_FALCO",
			"ruleSubtype": null,
			"matchedOnDefault": false,
			"fields": [{
				"key": "falco.rule",
				"value": "Read sensitive file untrusted"
			}],
			"falsePositive": false,
			"timestamp": 1518849310380639,
			"hostMac": "06:90:90:7f:15:ea",
			"isAggregated": false
		}]
	}]
}
</code></pre>
</noscript>

Note that time-related fields are formatted as 'microseconds since the epoch time', you will probably need to adapt this to the standard date format used by your engine.

For example, you can define a <a href="https://www.elastic.co/guide/en/elasticsearch/reference/current/pipeline.html" target="_blank">pipeline</a> and field mapping for Elasticsearch. Elasticsearch accepts a maximum time resolution of milliseconds, this pipeline will create an additional *estimestamp* field, preserving the original data:

    {
      "datetrim": {
        "description": "trim date to milliseconds",
        "processors": [
          {
            "script": {
              "lang": "painless",
              "source": "ctx.estimestamp = ctx.timestamp / 1000"
            }
          }
        ]
      }
    }

And now we define a mapping declaring the *estimestamp* field as a date in 'milliseconds since the epoch':

    {
      "mappings": {
        "event": {
          "properties": {
            "estimestamp":  {
              "type":   "date",
              "format": "epoch_millis"
            }
          }
        }
      }
    }

First, make sure that the WebHook is correctly configured and the data is flowing to your aggregation engine where it is being preprocessed and indexed.

Then, it's time to use all your data analysis expertise over that Docker / Kubernetes data processed by Sysdig as events, alerts and security policy violations.

## Elasticsearch / ELK dashboards for Kubernetes and Docker {#elasticsearchelkdashboardsforkubernetesanddocker}

Now you are ready to build your own dashboards in Elasticsearch / ELK using Kibana. These are some of the examples we built:

**Elasticsearch + Kibana, raw data search:**

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/sysdig_elasticsearch_kibana-1024x490.png" alt="sysdig elasticsearch kibana" width="1024" height="490" class="alignleft size-large wp-image-6635" />][4] 

First, we create a <a href="https://www.elastic.co/guide/en/kibana/current/tutorial-define-index.html" target="_blank">Kibana Index Pattern</a> and *Discover* the raw data to verify that: 
*   Source event fields are correctly mapped
*   We have the expected number of items and 
*   The timestamps are also correctly parsed by the engine.

**Elasticsearch + Kibana example Sysdig Monitor Dashboard:**

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/sysdig_elasticsearch_monitor-1024x570.png" alt="sysdig elasticsearch sysdig monitor events" width="1024" height="570" class="alignleft size-large wp-image-6636" />][5] 

In this example we have classified Sysdig Monitor alerts by severity and whether or not they are still active (pie graph), on the top-right table we are displaying the text information together with the alert name and HTTPS link to visualize directly on Sysdig Monitor. At the bottom, we have a time series graph segmented by Kubernetes namespace.

Sysdig events give you plenty of Docker / Kubernetes metadata to configure your container-specific data mining.

**Elasticsearch + Kibana example Sysdig Secure Dashboard:**

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/sysdig_elasticsearch_secure-1024x586.png" alt="Sysdig Elasticsearch secure" width="1024" height="586" class="alignleft size-large wp-image-6637" />][6] 

This dashboard is very similar to the last one, but in this case, using Sysdig Secure events. We also have a time distribution, severity / resolved pie and a data table, ordered by the severity of the security event.

## Splunk dashboards for Kubernetes and Docker {#splunkdashboardsforkubernetesanddocker}

**Splunk, raw data search:**

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/sysdig_splunk_search-1024x470.png" alt="Sysdig splunk search" width="1024" height="470" class="alignleft size-large wp-image-6641" />][7] 

First, we dump all the data choosing "All time" period and just the configured index *sysdigsecure* to check if the engine is correctly receiving and interpreting the JSON input stream.

**Splunk example Sysdig Monitor Dashboard:**

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/sysdig_splunk_monitor-1024x500.png" alt="Splunk Sysdig Monitor events" width="1024" height="500" class="alignleft size-large wp-image-6640" />][8] 

This is a prototype Sysdig Monitor dashboard, featuring an alert severity pie, a global state gauge (# of active alerts * severity), frequency of events time series and text information table at the bottom.

**Splunk example Sysdig Secure Dashboard:**

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/sysdig_splunk_secure-1024x499.png" alt="Sysdig Splunk Sysdig Secure" width="1024" height="499" class="alignleft size-large wp-image-6642" />][9] 

… and the Sysdig Secure dashboard with roughly the same information as the one we created for Elasticsearch.

## Further thoughts {#furtherthoughts}

*   We would love to know how you collect, store and visualize your Docker and Kubernetes events and alerts, are you using Elasticsearch / ELK, Splunk or something else?
*   Which other logging or tracing systems do you run alongside your monitoring tool?

Let us know through Twitter to <a href="https://twitter.com/sysdig" target="_blank">@Sysdig</a> or on <a href="https://slack.sysdigrp2rs.wpengine.com/" target="_blank">our Sysdig community Slack</a>!

PD: You can also get event data in and out Sysdig using the <a href="https://app.sysdigcloud.com/apidocs/" target="_blank">Sysdig REST API</a> or <a href="https://github.com/draios/python-sdc-client" target="_blank">Python libraries</a>.

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/sysdig_notification_channels.png
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/sysdig_elasticsearch_webhook.png
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/sysdig_splunk_webhook.png
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/sysdig_elasticsearch_kibana.png
 [5]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/sysdig_elasticsearch_monitor.png
 [6]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/sysdig_elasticsearch_secure.png
 [7]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/sysdig_splunk_search.png
 [8]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/sysdig_splunk_monitor.png
 [9]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2018/03/sysdig_splunk_secure.png