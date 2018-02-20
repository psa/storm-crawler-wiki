This document describes all configuration parameters that determine the behaviour of the crawler and all its components.

## Default configuration
The file [crawler-default.yaml](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/resources/crawler-default.yaml) lists the configuration elements presented below and provides a default value for them. This file is loaded automatically by the sub-classes of [ConfigurableTopology](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/ConfigurableTopology.java) and should not be modified. Instead we recommend that you provide a custom configuration file when launching a topology (see below).

## Custom configuration
The custom configuration file is expected to be in YAML format and can be passed as a command-line argument as `-conf <path_to_config_file>` to the Java call of your Main class (which normally would be a sub-class of [ConfigurableTopology](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/ConfigurableTopology.java)).

The values in the custom configuration file will override the ones provided in crawler-default.yaml and it does not need to contain all the values.

You can use `-conf <path_to_config_file>` more than once on the command line, which allows to separate the configuration files for instance between the generic configuration and the configuration of a specific resources. 

With Maven installed, you must first generate an uberjar:

``` sh
mvn clean package
```

before submitting the topology using the storm command:

```bash
storm jar path/to/allmycode.jar org.me.MyCrawlTopology -conf my-crawler-conf.yaml -local"
```
While deploying on a production Storm cluster, simply remove the `local` parameter.

Passing a configuration file is **mandatory**.
A sample configuration file can be found [here](https://github.com/DigitalPebble/storm-crawler/blob/master/archetype/src/main/resources/archetype-resources/crawler-conf.yaml).

## Configuration options
The following tables describe all available configuration options and their default values.
If one of the keys is not present in your YAML file, the default value will be taken.

### General options
| key                    | default value | description                            |
|------------------------|---------------|----------------------------------------|
| urlfilters.config.file | `urlfilters.json`| A JSON configuration file that defines URL filtering strategy. [Here is the default implementation](https://github.com/DigitalPebble/storm-crawler/blob/master/archetype/src/main/resources/archetype-resources/src/main/resources/urlfilters.json). Please also refer to [URLFilters](https://github.com/DigitalPebble/storm-crawler/wiki/URLFilters). **Note:** if you want to specify your own file you should give it a different name than `urlfilters.json`. For more information see [here](https://github.com/DigitalPebble/storm-crawler/issues/61)|
| parsefilters.config.file| `parsefilters.json` | The JSON configuration file that defines your ParseFilters. [Here](https://github.com/DigitalPebble/storm-crawler/blob/master/archetype/src/main/resources/archetype-resources/src/main/resources/parsefilters.json) is the default one. This influences the behavior of JSoupParserBolt and SiteMapParserBolt. **Note:** if you want to specify your own file you should give it a different name than `parsefilters.json`. For more information see [here](https://github.com/DigitalPebble/storm-crawler/issues/61) |

### Fetching and partitioning

| key                    | default value | description                            |
|------------------------|---------------|----------------------------------------|
| http.agent.name        | -             | A name to be part of the `User-Agent` request header for requests issued by the crawler |
| http.agent.version     | -             | A version to be part of the `User-Agent` request header for requests issued by the crawler |
| http.agent.description | -             | A description to be part of the `User-Agent` request header for requests issued by the crawler |
| http.agent.url         | -             | A URL to be part of the `User-Agent` request header for requests issued by the crawler (e.g. your Companies Homepage) |
| http.agent.email       | -             | An Email address to be part of the `User-Agent` request header for requests issued by the crawler |
| http.store.responsetime| `true`        | *not yet implemented* - whether or not to store the response time time in the Metadata |
| http.skip.robots       | `false`       | Generally ignore all robots.txt rules (not recommended) |
| http.proxy.host        | -             | A SOCKS HTTP proxy server to be used for all requests made by the crawler |
| http.proxy.port        | -             | The port of your SOCKS proxy server |
| http.timeout           | `10000`       | A connection timeout specified in milliseconds. Tuples that run into this timeout will be emitted with the status ERROR in the [StatusStream](https://github.com/DigitalPebble/storm-crawler/wiki/statusStream) |
| http.robots.agents     | `''`          | Comma separated additional user-agent strings to be used for the interpretation of the robots.txt. If left empty (default) than the robots.txt is interpreted with the value of `http.agent.name`|
| http.use.cookies       | `false`       | Use cookies from the repsonse in requests sent to direct child links. |
| http.content.limit     | `65536`       | - The maximum number of bytes for returned HTTP response bodies. Set `-1` to disable the limit.|
| partition.url.mode     | `byHost`      | Possible values are: `byHost`, `byDomain`, `byIP`. Defines how URLs are partitioned and by that routed to the FetcherBolt. For example `byIP` would lead to all tuples with a URL that is served by the same IP address to be always (for the lifetime of your topology) fetched by the same Storm task. This partitioning is important because it makes things like e.g. caching a robots.txt file for a specific domain very efficient. The value you specify here is being used to make use of Storms Field Grouping.|
| fetcher.queue.mode     | `byHost`      | _???_ Possible values are: `byHost`, `byDomain`, `byIP`. This parameter influences how FetchQueues are grouped inside the FetcherBolt. This influences the overall thread count and things like crawl delays (see below)|
| fetcher.max.crawl.delay| `30`          | The maximum number in seconds that will be accepted by [Crawl-delay](http://en.wikipedia.org/wiki/Robots_exclusion_standard#Crawl-delay_directive) directives in robots.txt files. If a page's robots.txt has a higher value than this, the tuple is emitted to the [StatusStream ](https://github.com/DigitalPebble/storm-crawler/wiki/statusStream) as an ERROR  |
| fetcher.threads.per.queue| `1`         | The default number of threads per queue. This can be overwritten for specific hosts/domains/IPs. See below |
| fetcher.maxThreads._host/domain/ip_  | `fetcher.threads. per.queue` | Overwrites the default value of `fetcher.threads.per.queue`. This is very useful if you have domains/hosts/IPs that you want to crawl more intensively (e.g. because they a lot of URLs emitted by your Spout |
| fetcher.server.delay   | `1`           | Defines delay between crawls in the same queue if no [Craw-delay](http://en.wikipedia.org/wiki/Robots_exclusion_standard#Crawl-delay_directive) is defines for this URL in the pages robots.txt. **Note:** For multi-threaded queues neither this value nor the one from the robots.txt will be honored. See `fetcher.server.min.delay`.|
| fetcher.server.min.delay| `0`          | _???_ Defines the delay between crawls in the same queue if a queue has > 1 thread. The Crawl-delay  declared in the robots.txt is ignored in this case and this value is taken. |
| fetcher.threads.number | `10`          | The number of threads that fetch pages from all queues concurrently. This threads does the actual work of downloading the page. Increase this to get more throughput at a cost of higher network, CPU and memory utilisation. Tweak this value carefully while looking at your system resources to find a value that works best for your hardware and network infrastructure. |
| redirections.allowed   | `true`        | If URL redirects are allowed or not. If set to true, the crawler will emit the targeted URL in the [StatusStream](https://github.com/DigitalPebble/storm-crawler/wiki/statusStream) with the status DISCOVERED |
| http.robots.403.allow  | `true`        | _???_ Defines what happens in the scenario where the request to the `robots.txt` file is being responded with a HTTP 403 (Forbidden). When set to `true` this means that the crawler would not be limited at all and freely crawl all pages of the domain. If set to `false` this means that the crawler would not fetch any of the pages for this domain. |
| protocols              | `http,https`  | The protocols to support. Each of them has a corresponding `<proto>.protocol.implementation` directive. Don't touch this unless you are implementing additional protocols to be supported. |
| http.protocol.implementation | `com.digitalpebble. storm.crawler.protocol. httpclient.HttpProtocol` | The [Protocol](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/protocol/Protocol.java) implementation for plain HTTP |
| https.protocol.implementation | `com.digitalpebble. storm.crawler.protocol. httpclient.HttpProtocol` | The [Protocol](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/protocol/Protocol.java) implementation for HTTP over SSL | 



### Indexing

The values below are used by sub-classes of `AbstractIndexerBolt`. Examples: [StdOut](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/indexing/StdOutIndexer.java), [ElasticSearch](https://github.com/DigitalPebble/storm-crawler/blob/master/external/elasticsearch/src/main/java/com/digitalpebble/storm/crawler/elasticsearch/bolt/IndexerBolt.java). These classes persist the outcome of your crawling process and receive tuples enriched with Metadata (with all information gathered by previous Bolts)

| key                    | default value | description                            |
|------------------------|---------------|----------------------------------------|
| indexer.md.filter      | -             | A YAML List of `key=value` strings that let you filter records that should be index based on Metadata of a tuple. If specified, only tuples that match the given filter are being indexed. This is just used by the helper method `AbstractIndexerBolt.filterDocument(Metadata)`. Using this method is on the responsibility of the implementing class. [Here is an example] (https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/indexing/StdOutIndexer.java#L56)|
| indexer.md.mapping     | -             | A YAML List of `key=value` strings that let you define a mapping of   fields that occur in the Metadata of a tuple to field-names for your persistence layer. The [AbstractIndexerBolt](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/indexing/AbstractIndexerBolt.java) provides a method names `filterMetadata(Metadata)` that sub-classes should use inside their `execute()` method in order to apply this mapping to the Metadata object. [Here](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/storm/crawler/indexing/StdOutIndexer.java#L72) is an example. |
| indexer.text.fieldname | -             | The fieldname that should be used to index the content of HTML body. The usage of this is again in the responsibility of the class that extends `AbstractIndexerBolt`. The value of this can be accessed using the protected method `fieldNameForText()`. [Here](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/indexing/StdOutIndexer.java#L64) is an example. | 
| indexer.url.fieldname  | -             | Same as above - `indexer.text.fieldname` just for the URL Field |

### Status persistence
_This refers to persisting the status of a URL (e.g. ERROR, DISCOVERED etc.) along with a something like a `nextFetchDate` that is being calculated by a [Scheduler](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/persistence/DefaultScheduler.java)_

| key                    | default value | description                            |
|------------------------|---------------|----------------------------------------|
| status.updater.use.cache | `true`      | Using this cache helps to prevent persisting the same URLs over and over again. The `store()` method of your implementation of `AbstractStatusUpdaterBolt` ([example](https://github.com/DigitalPebble/storm-crawler/blob/master/external/elasticsearch/src/main/java/com/digitalpebble/stormcrawler/elasticsearch/persistence/StatusUpdaterBolt.java#L98)) is only called if a URL does not already exist in the cache. This is a simple but efficient improvement to avoid re-persisting e.g. the same internal links over and over again.|
| status.updater.cache.spec | `maximumSize=10000, expireAfterAccess=1h` | A [cache specification](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/cache/CacheBuilderSpec.html) string that defines the size and behavior of the above cache.|
| fetchInterval.default   | `1440`     | In minutes - how to schedule re-visits of pages. 1 Day by default.  This is used by the [DefaultScheduler](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/persistence/DefaultScheduler.java). If you need customized scheduling logic, just implement your own [Scheduler](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/storm/crawler/persistence/Scheduler.java). **Note:** the Scheduler class is not yet configurable. See or update (this issue)[https://github.com/DigitalPebble/storm-crawler/issues/104] if you need this bahvior. Should be quite easy to make the implemetation class configurable.| 
| fetchInterval.fetch.error | `120`     | In minutes - how often to re-visit pages with a fetch error. Every two hours by default. Identified by tuples in the (StatusStream)[https://github.com/DigitalPebble/storm-crawler/wiki/statusStream] with the state of `FETCH_ERROR` |
| fetchInterval.error     | `44640`     | In minutes - how often to re-visit pages with an error (HTTP 4XX or 5XX). Every month by default. Identified by tuples in the (StatusStream)[https://github.com/DigitalPebble/storm-crawler/wiki/statusStream] with the state of `ERROR`. |


### Parsing
| key                    | default value | description                            |
|------------------------|---------------|----------------------------------------|
| parser.emitOutlinks    | `true`        |  Whether or not to emit outgoing links found in the parsed HTML document to the [StatusStrean](https://github.com/DigitalPebble/storm-crawler/wiki/statusStream) as `DISCOVERED`. Your [URL Filters](https://github.com/DigitalPebble/storm-crawler/wiki/URLFilters) are applied to outgoing links before they are emitted. This option being `true` is crucial if you are building a recursive crawler. |
| track.anchors          | `true`        | Whether or not to add the anchor text (can be > 1) of (filtered) outgoing links with the key `anchors` to the Metadata of a tuple. 

### Metadata
| key                    | default value | description                            |
|------------------------|---------------|----------------------------------------|
| metadata.track.path    | `true`        | Whether or not to track the URL path of outgoing links (all URLs that the crawler crawled to find this link) in the Metadata. The Metadata field name for this is `url.path`. It's a list of URLs that represent the crawl path (how did the crawler find this page). |
| metadata.track.depth   | `true`        | Whether or not to track the depth of a crawled URL. This is a simple counter that is being tracked for outgoing links in the Metadata and incremented by `1` for every page that was crawled to find a specific link. This can be useful to let your Spout decide/sort which URLs to emit based on their depth. You could use this to influence the behavior of your recursive crawl (e.g. prefer pages with a low `depth` count).|