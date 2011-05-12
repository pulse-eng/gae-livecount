# About

Livecount is a fast and simple implementation of real-time counters for Google AppEngine.  The goal is to support the counting of real-time events such as impressions, clicks, conversions, and other user actions and to do so in an efficient and scalable way.  Since Livecount increments counts in memory, it is  almost always faster than basic sharded counters, potentially trading off some consistency and durability.  To ensure maximum consistency and durability, by default Livecount attempts to asynchronously write each update to the datastore.

Livecount counters are extremely simple to use.  Just import the module and call load_and_increment_counter() with a key name and increment delta.  Livecount also supports arbitrary string keys allowing more complex character-delimited hierarchal key and supports the use of appengine's memcache namespaces maintain separate counters with the same key.

# Getting started

To try Livecount:

1. Make sure you have the latest version of the Google AppEngine python sdk
2. git clone git://github.com/gregbayer/gae-livecount.git
3. Point the appengine launcher at the livecount directory and start the project locally
4. Navigate to http://localhost:8080/livecount/counter_admin
5. Choose to login as an administrator

To add Livecount to your project:

1. git clone git://github.com/gregbayer/gae-livecount.git
2. Add entries to your app.yaml and queue.yaml based on included files.
3. Copy the livecount directory into your appengine project
4. Include "from counters import counter" at the top of you python file
5. Call counter.load_and_increment_counter(...) as in example.py

# Performance and CAP tradeoffs

Livecount makes extensive use of Google AppEngine's memcache implementation.  This allows maximum performance, while leaving counts vulnerable to a memcache eviction.  This risk is mitigated by using Google AppEngine's taskqueue to asynchronous write the value of a count back to the datastore after any change.  To avoid unnecessary overhead, a new asynchronous worker is only created if one is not already waiting to update a giving counter's value.  If one is already in the queue, it simply writes back the most recent count.  The resulting risk of lost counts is minimal and acceptable for many real-time counter use cases.

If writeback loads are higher than desired on frequently updated counters, Livecount includes the option to delay writebacks until a given batch size is reached.  This allows the developer to select the appropriate tradoff between performance and tolerance for lost counts each time a counter is incremented.

# Scalability Limiatations

One factor that affects Livecount's scalability is the rate at which appengine task queues can writeback counter updates (xxx tasks/second per queue). See [Quotas and Limits for Push Queues](http://code.google.com/appengine/docs/python/taskqueue/overview-push.html) for more information.  To effectively increase this limit, Livecount could be extended to use more than one named queue. 

The maximum number of queues could also be reached.  At this point, a different writeback strategy should be used.  One option would be to batch updates for several different counters in one memcache object and only create a writeback worker when the batch size is reached.

# Usage at Pulse

Livecounts has been used extensively at [Pulse News](http://pulsene.ws) for counting everything from share and click events to mobile client A/B test results. 

# Other Projects and Information

Livecount is by no means the first or only attempt at real-time counters.  Here are a few interesting links:

* [Sharded Counters](http://code.google.com/appengine/articles/sharding_counters.html)
* [Brett Slatkin's Google I/O talk](http://sites.google.com/site/io/building-scalable-web-applications-with-google-app-engine)
* [Effective memcache](http://code.google.com/appengine/articles/scaling/memcache.html)
* [Google Cookbook](http://appengine-cookbook.appspot.com/recipe/high-concurrency-counters-without-sharding/)
* [fastpageviews](http://code.google.com/p/fastpageviews/)
* [Rainbird (Kevin Weil / Twitter)](http://www.slideshare.net/kevinweil/rainbird-realtime-analytics-at-twitter-strata-2011)


