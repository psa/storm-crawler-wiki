**Storm-crawler** is a SDK for building web crawlers with [Apache Storm](http://storm.apache.org). The project is under Apache license v2 and consists of a collection of reusable resources and components. 

The aims of storm-crawler is to help build web crawlers that are \:
* scalable
* low latency
* easy to extend
* polite yet efficient

Storm-crawler is a library and collection of resources that developers can leverage to build their own crawlers. The good news is that doing is be pretty straightforward. As explained on [Getting Started](http://stormcrawler.net/getting-started/), you can use the Maven archetype to bootstrap a new project and tweak it to your heart's content.

Apart from the core components, we also provide some [external resources](https://github.com/DigitalPebble/storm-crawler/tree/master/external) that you can reuse in your project, like for instance our spout and bolts for [ElasticSearch](https://www.elastic.co/) or a ParserBolt which uses [Apache Tika](http://tika.apache.org) to parse various document formats.

**Storm-crawler** is perfectly suited to use cases where the URL to fetch and parse come as streams but is also a good solution for large-scale recursive crawls, particularly where low latency is required. The project is used in production by [various companies](https://github.com/DigitalPebble/storm-crawler/wiki/Powered-By) and is actively developed and maintained.

The [[Presentations]] page contains links to some recent presentations made about this project.