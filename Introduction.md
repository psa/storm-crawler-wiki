**Storm-crawler** is a SDK for building web crawlers with [Apache Storm](http://storm.apache.org). The project is under Apache license v2 and consists of a collection of reusable resources and components. 

The aims of storm-crawler is to help build web crawlers that are \:
* scalable
* low latency
* easy to extend
* polite yet efficient

Unlike other crawlers like [Apache Nutch](http://nutch.apache.org) or [Scrapy](http://scrapy.org/), storm-crawler is not a ready-to-use software, but a library that developers can leverage to build their own crawlers. The good news is that doing so can be pretty straightforward. Often, all you'll have to do will be to declare storm-crawler as a Maven dependency, write your own Topology class (tip \: you can extend [ConfigurableTopology](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/ConfigurableTopology.java)), reuse the components provided by the project and maybe write a couple of custom ones for your own secret sauce. A bit of tweaking to the [Configuration](https://github.com/DigitalPebble/storm-crawler/wiki/Configuration) and off you go!

Apart from the core components, we also provide some [external resources](https://github.com/DigitalPebble/storm-crawler/tree/master/external) that you can reuse in your project, like for instance our spout and bolts for [ElasticSearch](https://www.elastic.co/) or a ParserBolt which uses [Apache Tika](http://tika.apache.org) to parse various document formats.

**Storm-crawler** is perfectly suited to use cases where the URL to fetch and parse come as streams but is also a good solution for large scale recursive crawls, particularly where low latency is required. The project is used in production by [various companies](https://github.com/DigitalPebble/storm-crawler/wiki/Powered-By) and is actively developed and maintained.

The [[Presentations]] page contains links to some recent presentations made about this project.