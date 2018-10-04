---
ID: 1508
post_title: 'MongoDB Showdown: Aggregate vs Map-Reduce'
author: Luca Marturana
post_excerpt: ""
layout: post
permalink: >
  https://sysdigrp2rs.wpengine.com/blog/mongodb-showdown-aggregate-vs-map-reduce/
published: true
post_date: 2015-03-12 06:00:58
---
<a href="http://www.mongodb.org/" target="_blank">MongoDB</a> is widely regarded as the leading NoSQL database technology. Developers love it because MongoDB is schema-less and it helps them build applications much faster than developing with relational databases, and it scales very well in production. MongoDB also has many features that make it unique from other noSQL databases, including things like geospatial queries and map-reduce operations.

Map-reduce is a common pattern when working with Big Data - it's a way to extract info from a huge dataset. But now, starting with version 2.2, MongoDB includes a new feature called <a href="http://docs.mongodb.org/manual/core/aggregation-introduction/" target="_blank">Aggregation</a> framework. Functionality-wise, Aggregation is equivalent to map-reduce but, on paper, it promises to be much faster.

I thought it might be fun to test this claim ;)

So I spent some time utilizing Sysdig Monitor to compare the performance of Aggregate vs map-reduce (read more about Sysdig Monitor new <a href="https://sysdigrp2rs.wpengine.com/sysdig-cloud-now-supports-mongodb-new-built-in-views/" target="_blank">MongoDB support here</a>). The rest of this post will reveal my findings. The process of discovery itself was interesting though, so let's start from the start, and go through it together...

## Setup

First of all, I needed some test data for our queries. I can create it using the ruby <a href="https://github.com/stympy/faker" target="_blank">Faker</a> library. For example, this scripts creates 3 million simulated customer entries:

    require 'faker'
    require 'mongo'
    
    include Mongo
    
    client = MongoClient.new(ENV['MONGODB'], 27017)
    db = client["test"]
    collection = db["customers"]
    
    3000000.times do
    	collection.insert({
    			:first_name => Faker::Name.first_name,
    			:last_name => Faker::Name.last_name,
    			:city => Faker::Address.city,
    			:country_code => Faker::Address.country_code,
    			:orders_count => Random.rand(10)+1
    		})
    end
    

Now let's exclude the script:

    MONGODB=<server_address> ruby filldata.rb
    

At this point, I can create a script that simulates an app that uses this data to get the sum of the orders grouped by country code:

    require 'mongo'
    
    include Mongo
    
    client = MongoClient.new(ENV['MONGODB'], 27017)
    db = client["test"]
    collection = db["customers"]
    
    loop do
        collection.aggregate( [
                            { "$match" => {}},
                            { "$group" => {
                                          "_id" => "$country_code",
                                          "orders_count" => { "$sum" => "$orders_count" }
                                        }
                           }
                   ])
          collection.map_reduce("function() { emit(this.country_code, this.orders_count) }",
                      "function(key,values) { return Array.sum(values) }",  { :out => { :inline => true }, :raw => true});
    end
    

The *collection.aggregate* and the *collection.map_reduce* queries in the script are doing the exactly the same thing, they just leverage a different underlying MongoDB facility.

Let's run the script:

    MONGODB=<server_address> ruby query.rb

Our servers are already instrumented using Sysdig Monitor's agent. It's a very lightweight agent that listens to all system events, collecting metrics and reporting them back to Sysdig Monitor's services. It can be installed with a single command:

    curl -s https://s3.amazonaws.com/download.draios.com/stable/install-agent | sudo bash -s

## Visualizing MongoDB Activity

Sysdig Monitor is also able to get AWS instance names so I can easily identify our MongoDB server:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/03/Screeny-Shot-Feb-19-2015-9.35.34-AM-1024x625.png" alt="MongoDB Server" width="1024" height="625" class="alignnone size-large wp-image-1513" />][1]

By clicking on it, I'm able to drill down into the rich set of views and metrics that Sysdig Monitor collects with no additional instrumentation. In particular, I'm going to pick the *MongoDB Overview* view.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/03/Screeny-Shot-Feb-19-2015-11.25.10-AM-1024x460.png" alt="MongoDB Overview" width="1024" height="460" class="alignnone size-large wp-image-1518" />][2]

Here's one of the charts in the view:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/03/Screeny-Shot-Feb-19-2015-9.38.06-AM-1024x432.png" alt="Screeny Shot Feb 19, 2015, 9.38.06 AM" width="1024" height="432" class="alignnone size-large wp-image-1514" />][3]

I can clearly see two sets of periodic spikes, with very different magnitude. I bet the higher spikes are generated by the map-reduce queries, while the lower ones are caused by the Aggregate queries. To find out, let's dig deeper by creating a custom chart that shows the MongoDB request time grouped by query type.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/03/2015-02-19-14_29_38.gif" alt="Map-Reduce vs Aggregate" width="1024" height="294" class="alignnone size-large wp-image-1510" />][4]

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/03/Screeny-Shot-Feb-19-2015-9.57.52-AM-1024x646.png" alt="Aggregate v Map-Reduce Pin to Dash" width="1024" height="646" class="alignnone size-large wp-image-1516" />][5]

And now let's move this cart into a dashboard.

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/03/Screeny-Shot-Feb-19-2015-9.58.12-AM-1024x646.png" alt="Aggregate vs Map Reduce - Pin to Dashboard & Open" width="1024" height="646" class="alignnone size-large wp-image-1517" />][6]

Let's also include a CPU usage chart in the same dashboard, so I can correlate MongoDB activity with CPU usage.

With our dashboard opened, I am now going to execute both queries. Let's take a look at the results:

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/03/Screeny-Shot-Feb-19-2015-2.14.46-PM-1024x644.png" alt="Avg MongoDB request time for Aggregate vs Map Reduce" width="1024" height="644" class="alignnone size-large wp-image-1511" />][7]

[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/03/Screeny-Shot-Feb-19-2015-2.14.51-PM-1024x644.png" alt="CPU% over time for MongoDB" width="1024" height="644" class="alignnone size-large wp-image-1512" />][8]

Sysdig Monitor's one-second granularity allows us to very precisely correlate the spikes in CPU utilization with the execution of the queries. You can clearly see that map-reduce uses way more CPU than Aggregate. Also, the duration of the CPU spikes matches quite perfectly with the measured response time, proving that the CPU is used to serve the queries.

To better quantify the speed difference I can plot a Top 10 chart with an average on a timespan of 5 minutes. These are the results:   
[<img src="https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/03/Screeny-Shot-Feb-19-2015-9.52.13-AM-1024x630.png" alt="Top Avg Request Time - Aggregate vs Map Reduce" width="1024" height="630" class="alignnone size-large wp-image-1515" />][9]

Aggregate is about 6x faster than map-reduce! [tweet_box design="default" float="none"] #Monitoring #MongoDB performance: aggregate is about 6x faster than map-reduce [/tweet_box] 

## Conclusion

The obvious conclusion is: if you are sending map-reduce queries to your Mongo backend and are concerned about performance, you should try switching to the Aggregation framework as soon as possible.

More importantly: running tests like this can help you and your organization become more data-driven when it comes to making design decisions for your application environment.

Curious to see how your own MongoDB deployment performs? <a href="https://sysdigrp2rs.wpengine.com/landing-pitch3/?utm_source=web&utm_medium=blog&utm_campaign=mongodbaggvsmr031115" target="_blank">Sign up for a 15 days free trial</a>, install the Sysdig Monitor agents, and you'll be running your own MongoDB experiments in just a matter of minutes.

 [1]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/03/Screeny-Shot-Feb-19-2015-9.35.34-AM.png
 [2]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/03/Screeny-Shot-Feb-19-2015-11.25.10-AM.png
 [3]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/03/Screeny-Shot-Feb-19-2015-9.38.06-AM.png
 [4]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/03/2015-02-19-14_29_38.gif
 [5]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/03/Screeny-Shot-Feb-19-2015-9.57.52-AM.png
 [6]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/03/Screeny-Shot-Feb-19-2015-9.58.12-AM.png
 [7]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/03/Screeny-Shot-Feb-19-2015-2.14.46-PM.png
 [8]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/03/Screeny-Shot-Feb-19-2015-2.14.51-PM.png
 [9]: https://sysdigrp2rs.wpengine.com/wp-content/uploads/2015/03/Screeny-Shot-Feb-19-2015-9.52.13-AM.png