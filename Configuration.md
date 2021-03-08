This document describes all configuration parameters that determine the behaviour of the crawler and all its components.

# Table of Contents

* [Default configuration](#default-configuration)
* [Custom configuration](#custom-configuration)
* [Configuration options](#configuration-options)
  * [Fetching and partitioning](#fetching-and-partitioning)
  * [Protocol](#protocol)
  * [Indexing](#indexing)
  * [Status persistence](#status-persistence)
  * [Parsing](#parsing)
  * [Metadata](#metadata)


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

### Fetching and partitioning

Configuration for
[Bolts](https://github.com/DigitalPebble/storm-crawler/tree/master/core/src/main/java/com/digitalpebble/stormcrawler/bolt)
handling the fetching and partitioning of data. Some keys overlap with classes
in
[Protocol](https://github.com/DigitalPebble/storm-crawler/tree/master/core/src/main/java/com/digitalpebble/stormcrawler/protocol)
as well, although most of that configuration can be found in the
[protocol](#protocol) section.

| key                    | default value | description                            |
|------------------------|---------------|----------------------------------------|
| fetcher.max.crawl.delay | `30`          | The maximum number in seconds that will be accepted by [Crawl-delay](http://en.wikipedia.org/wiki/Robots_exclusion_standard#Crawl-delay_directive) directives in robots.txt files.  If the crawl-delay exceeds this value the behavior depends on the value of `fetcher.max.crawl.delay.force`. |
| fetcher.max.crawl.delay.force | false   | Configures the behavior of fetcher if the robots.txt crawl-delay exceeds `fetcher.max.crawl.delay`. If false: the tuple is emitted to the [StatusStream ](https://github.com/DigitalPebble/storm-crawler/wiki/statusStream) as an ERROR. If true: the queue delay is set to `fetcher.max.crawl.delay`. |
| fetcher.max.queue.size | `-1`          | The maximum length of the queue used to store items to be fetched by the FetcherBolt. A setting of `-1` sets the length to `Integer.MAX_VALUE` |
| fetcher.max.throttle.sleep| `-1`       | The maximum amount of time to wait between fetches, if the time to wait exceeds this maximum that item will be sent to the back of the queue. Used in `SimpleFetcherBolt`. `-1` to disable. |
| fetcher.max.urls.in.queues | `-1`      | Limits the number of URLs that can be stored in a fetch queue. This includes the URLs that are currently being fetched. `-1` disables the limit. |
| fetcher.maxThreads._host/domain/ip_  | `fetcher.threads.per.queue` | Overwrites the default value of `fetcher.threads.per.queue`. This is very useful if you have domains/hosts/IPs that you want to crawl more intensively (e.g. because they a lot of URLs emitted by your Spout |
| fetcher.metrics.time.bucket.secs | `10` | Metrics events will be emitted to the system stream every _value_ seconds. These events can be read by registering a `MetricConsumer` in the topology. |
| fetcher.queue.mode     | `byHost`      | Possible values are: `byHost`, `byDomain`, `byIP`. This parameter influences how FetchQueues are grouped inside the FetcherBolt. This influences the overall thread count and things like crawl delays (see below) |
| fetcher.server.delay   | `1`           | Defines delay between crawls in the same queue if no [Craw-delay](http://en.wikipedia.org/wiki/Robots_exclusion_standard#Crawl-delay_directive) is defines for this URL in the pages robots.txt. **Note:** For multi-threaded queues neither this value nor the one from the robots.txt will be honored. See `fetcher.server.min.delay`. |
| fetcher.server.delay.force| false      | Defines the behavior of fetcher when the crawl-delay in the robots.txt is smaller than the value configured in `fetcher.server.delay`. If false the shorter crawl-delay from the robots.txt is used. If true the longer configured delay is forced. |
| fetcher.server.min.delay| `0`          | Defines the delay between crawls in the same queue if a queue has > 1 thread (`fetcher.server.delay` is use otherwise). The Crawl-delay declared in the robots.txt is ignored in this case and this value is taken. |
| fetcher.threads.number | `10`          | The number of threads that fetch pages from all queues concurrently. This threads does the actual work of downloading the page. Increase this to get more throughput at a cost of higher network, CPU and memory utilisation. Tweak this value carefully while looking at your system resources to find a value that works best for your hardware and network infrastructure. |
| fetcher.threads.per.queue | `1`        | The default number of threads per queue. This can be overwritten for specific hosts/domains/IPs. See below |
| fetcher.timeout.queue  |  `-1`         | The maximum time in seconds that an item can wait in the queue. `-1` disables the timeout |
| fetcherbolt.queue.debug.filepath | ""  | The Path to a debug log, e.g. `/tmp/fetcher-dump-{port}`. The content of the queues will be dumped to the logs. The port number must match the one used by the FetcherBolt instance. |
| http.agent.description | -             | A description to be part of the `User-Agent` request header for requests issued by the crawler |
| http.agent.email       | -             | An Email address to be part of the `User-Agent` request header for requests issued by the crawler |
| http.agent.name        | -             | A name to be part of the `User-Agent` request header for requests issued by the crawler |
| http.agent.url         | -             | A URL to be part of the `User-Agent` request header for requests issued by the crawler (e.g. your Companies Homepage) |
| http.agent.version     | -             | A version to be part of the `User-Agent` request header for requests issued by the crawler |
| http.basicauth.password | -             | Password associated with the property `http.basicauth.user` for the Basic Authentication |
| http.basicauth.user    | -             | A user used for the Basic Authentication implemented in HTTPClient protocole |
| http.content.limit     | `-1`       | The maximum number of bytes for returned HTTP response bodies. By default no limit is applied. In the generated archetype a limit of `65536` is present.|
| http.protocol.implementation | `com.digitalpebble.stormcrawler.protocol.httpclient.HttpProtocol` | The [Protocol](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/protocol/Protocol.java) implementation for plain HTTP |
| http.proxy.host        | -             | A HTTP proxy server to be used for all requests made by the crawler |
| http.proxy.pass        | -             | Password to use for HTTP Proxy basic authentication |
| http.proxy.port        | `8080`        | The port of your HTTP proxy server |
| http.proxy.user        | -             | Username to use for HTTP Proxy basic authentication |
| http.robots.403.allow  | `true`        | Defines what happens when a request for `robots.txt` is responded to with HTTP 403 (Forbidden). When set to `true` the crawler will crawl all pages of the domain. If set to `false` the crawler will not fetch any of the pages of this domain. |
| http.robots.agents     | `''`          | Comma separated additional user-agent strings to be used for the interpretation of the robots.txt. If left empty (default) than the robots.txt is interpreted with the value of `http.agent.name`|
| http.robots.file.skip  | `false`       | **1.17 and later, replaces http.skip.robots** Ignore robots.txt rules (not recommended) |
| http.skip.robots       | `false`       | **1.16 and earlier, replaced by http.robots.file.skip** Ignore robots.txt rules (not recommended) |
| http.store.headers     | `false`       |
| http.store.responsetime | `true`        | *not yet implemented* - whether or not to store the response time time in the Metadata |
| http.timeout           | `10000`       | A connection timeout specified in milliseconds. Tuples that run into this timeout will be emitted with the status ERROR in the [StatusStream](https://github.com/DigitalPebble/storm-crawler/wiki/statusStream) |
| http.use.cookies       | `false`       | Use cookies from the response in requests sent to direct child links. |
| https.protocol.implementation | `com.digitalpebble.stormcrawler.protocol.httpclient.HttpProtocol` | The [Protocol](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/protocol/Protocol.java) implementation for HTTP over SSL |
| partition.url.mode     | `byHost`      | Possible values are: `byHost`, `byDomain`, `byIP`. Defines how URLs are partitioned and by that routed to the FetcherBolt. For example `byIP` would lead to all tuples with a URL that is served by the same IP address to be always (for the lifetime of your topology) fetched by the same Storm task. This partitioning is important because it makes things like e.g. caching a robots.txt file for a specific domain very efficient. The value you specify here is being used to make use of Storms Field Grouping. |
| protocols              | `http,https`  | The protocols to support. Each of them has a corresponding `<proto>.protocol.implementation` directive. Don't touch this unless you are implementing additional protocols to be supported. |
| redirections.allowed   | `true`        | If URL redirects are allowed or not. If set to true, the crawler will emit the targeted URL in the [StatusStream](https://github.com/DigitalPebble/storm-crawler/wiki/statusStream) with the status DISCOVERED |
| sitemap.discovery      | `false`       | Enable automatic discovery of site maps. |

### Protocol

Configuration for the
[Protocol](https://github.com/DigitalPebble/storm-crawler/tree/master/core/src/main/java/com/digitalpebble/stormcrawler/protocol)
implementations. Note that some of configuration for these modules is shared
and also found in [Fetching and partitioning](#fetching-and-partitioning)

| key                       | default value | description                            |
|---------------------------|---------------|----------------------------------------|
| cacheConfigParamName      | `maximumSize=10000,expireAfterWrite=6h` | `CacheBuilder` configuration for the robots cache in `RobotRulesParser` |
| errorcacheConfigParamName | `maximumSize=10000,expireAfterWrite=1h` | `CacheBuilder` configuration for the error cache in `RobotRulesParser` |
| file.encoding             | `UTF-8`       | The encoding of files read by `FileProtocol` |
| http.accept            | -             | [HTTP Accept](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept) headers to send with connections using `HttpProtocol`. |
| http.accept.language   | -             | [HTTP Accept-Language](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Language) headers to send with connections using `HttpProtocol`. |
| http.content.partial.as.trimmed | `false` | If `true`, tells [OKHTTP](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/protocol/okhttp/HttpProtocol.java) to accept partially fetched content and mark it as trimmed content. Sets `TrimmedContentReason` to `DISCONNECT` |
| http.trust.everything | `true`            | If `true`, [OKHTTP](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/protocol/okhttp/HttpProtocol.java) should trust all SSL/TLS connections. |
| navigationfilters.config.file | -         | A JSON configuration pointing to a class which extends `NavigationFilter`. See the [dynamic content blog post](http://digitalpebble.blogspot.com/2017/04/crawl-dynamic-content-with-selenium-and.html) for details. |
| selenium.addresses        | -             | A list of addresses to WebDriver servers |
| selenium.capabilities     | -             | A map containing desired [WebDriver capabilities](https://github.com/SeleniumHQ/selenium/wiki/DesiredCapabilities). JavaScript is always enabled. |
| selenium.delegated.protocol | -           | A string pointing to an implementation of a class which implements `com.digitalpebble.stormcrawler.protocol`. This is called by `DelegatorRemoteDriverProtocol` if the incoming URL does not have `protocol.use.selenium` in its metadata. This allows using Selenium for a subset of the crawl. |
| selenium.implicitlyWait   | `0`           | The [WebDriver timeout](https://w3c.github.io/webdriver/#timeouts) for the element location strategy to attempt to find elements. |
| selenium.instances.num    | `1`           | The number of instances to create per WebDriver connection (each item in `selenium.addresses`). |
| selenium.pageLoadTimeout  | `-1`          | The [WebDriver timeout](https://w3c.github.io/webdriver/#timeouts) for attempting a page navigation load. |
| selenium.setScriptTimeout | `0`           | The [WebDriver timeout](https://w3c.github.io/webdriver/#timeouts) for executing a [WebDriver script](https://w3c.github.io/webdriver/#executing-script). If NULL, there is no limit. |
| topology.message.timeout.secs | `-1`      | Seconds [OKHTTP](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/protocol/okhttp/HttpProtocol.java) will wait for a page to be fetched |

### Indexing

The values below are used by sub-classes of `AbstractIndexerBolt`. Examples: [StdOut](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/indexing/StdOutIndexer.java), [ElasticSearch](https://github.com/DigitalPebble/storm-crawler/blob/master/external/elasticsearch/src/main/java/com/digitalpebble/storm/crawler/elasticsearch/bolt/IndexerBolt.java). These classes persist the outcome of your crawling process and receive tuples enriched with Metadata (with all information gathered by previous Bolts)

| key                    | default value | description                            |
|------------------------|---------------|----------------------------------------|
| indexer.md.filter      | -             | A YAML List of `key=value` strings that let you filter records that should be index based on Metadata of a tuple. If specified, only tuples that match the given filter are being indexed. This is just used by the helper method `AbstractIndexerBolt.filterDocument(Metadata)`. Using this method is on the responsibility of the implementing class. [Here is an example] (https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/indexing/StdOutIndexer.java#L56) |
| indexer.md.mapping     | -             | A YAML List of `key=value` strings that let you define a mapping of   fields that occur in the Metadata of a tuple to field-names for your persistence layer. The [AbstractIndexerBolt](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/indexing/AbstractIndexerBolt.java) provides a method names `filterMetadata(Metadata)` that sub-classes should use inside their `execute()` method in order to apply this mapping to the Metadata object. [Here](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/storm/crawler/indexing/StdOutIndexer.java#L72) is an example. |
| indexer.text.fieldname | -             | The fieldname that should be used to index the content of HTML body. The usage of this is again in the responsibility of the class that extends `AbstractIndexerBolt`. The value of this can be accessed using the protected method `fieldNameForText()`. [Here](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/indexing/StdOutIndexer.java#L64) is an example. |
| indexer.url.fieldname  | -             | Same as above - `indexer.text.fieldname` just for the URL Field |

### Status persistence

This refers to persisting the status of a URL (e.g. ERROR, DISCOVERED etc.) along with a something like a `nextFetchDate` that is being calculated by a [Scheduler](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/persistence/DefaultScheduler.java)

| key                    | default value | description                            |
|------------------------|---------------|----------------------------------------|
| fetchInterval.default   | `1440`     | In minutes - how to schedule re-visits of pages. 1 Day by default.  This is used by the [DefaultScheduler](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/persistence/DefaultScheduler.java). If you need customized scheduling logic, just implement your own [Scheduler](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/storm/crawler/persistence/Scheduler.java). **Note:** the Scheduler class is not yet configurable. See or update (this issue)[https://github.com/DigitalPebble/storm-crawler/issues/104] if you need this bahvior. Should be quite easy to make the implemetation class configurable. |
| fetchInterval.error     | `44640`     | In minutes - how often to re-visit pages with an error (HTTP 4XX or 5XX). Every month by default. Identified by tuples in the (StatusStream)[https://github.com/DigitalPebble/storm-crawler/wiki/statusStream] with the state of `ERROR`. |
| fetchInterval.fetch.error | `120`     | In minutes - how often to re-visit pages with a fetch error. Every two hours by default. Identified by tuples in the (StatusStream)[https://github.com/DigitalPebble/storm-crawler/wiki/statusStream] with the state of `FETCH_ERROR` |
| status.updater.cache.spec | `maximumSize=10000, expireAfterAccess=1h` | A [cache specification](http://docs.guava-libraries.googlecode.com/git/javadoc/com/google/common/cache/CacheBuilderSpec.html) string that defines the size and behavior of the above cache. |
| status.updater.use.cache | `true`      | Using this cache helps to prevent persisting the same URLs over and over again. The `store()` method of your implementation of `AbstractStatusUpdaterBolt` ([example](https://github.com/DigitalPebble/storm-crawler/blob/master/external/elasticsearch/src/main/java/com/digitalpebble/stormcrawler/elasticsearch/persistence/StatusUpdaterBolt.java#L98)) is only called if a URL does not already exist in the cache. This is a simple but efficient improvement to avoid re-persisting e.g. the same internal links over and over again. |

### Parsing

Configures parsing of fetched text and the handling of discovered URIs

| key                    | default value | description                            |
|------------------------|---------------|----------------------------------------|
| collections.file       | `collections.json` | The name of the configuration for the [CollectionTagger](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/parse/filter/CollectionTagger.java)
| collections.key        | `collections` | Tags will be stored in the metadata with this as the key. If there is not collections key in the JSON configuration, `Collections` filter reads the key from the main configuration. |
| feed.filter.hours.since.published | -1 | When a link is found by `FeedParserBolt`, discard it if it has a published time older than _value_ hours. |
| feed.sniffContent      | `false`       | If the metadata doesn't already indicate that the page is a feed, tells `FeedParserBolt` to sniff the content type metadata and the first part of the file to see if it can detect a feed. This will only work if the server returns `rss+xml` or has `<rss ` in the first few bytes of the content. |
| parsefilters.config.file | -           | Path to a configuration file for `ParseFilters`. The contents of which are described in the [ParseFilters](https://github.com/DigitalPebble/storm-crawler/wiki/ParseFilters) wiki page. |
| parsefilters.config.file| `parsefilters.json` | The JSON configuration file that defines your ParseFilters. [Here](https://github.com/DigitalPebble/storm-crawler/blob/master/archetype/src/main/resources/archetype-resources/src/main/resources/parsefilters.json) is the default one. This influences the behavior of JSoupParserBolt and SiteMapParserBolt. **Note:** if you want to specify your own file you should give it a different name than `parsefilters.json`. For more information see [here](https://github.com/DigitalPebble/storm-crawler/issues/61) |
| parser.emitOutlinks    | `true`        | Whether or not to emit outgoing links found in the parsed HTML document to the [StatusStrean](https://github.com/DigitalPebble/storm-crawler/wiki/statusStream) as `DISCOVERED`. Your [URL Filters](https://github.com/DigitalPebble/storm-crawler/wiki/URLFilters) are applied to outgoing links before they are emitted. This option being `true` is crucial if you are building a recursive crawler. |
| parser.emitOutlinks.max.per.page    | -1 | limits the number of links sent from a page |
| textextractor.exclude.tags | ""        | A list of HTML tags that should be ignored when the `TextExtractor` is searching for text. |
| textextractor.include.pattern | ""     | A list of regex patterns for `TextExtractor` to match text against. Only text matching the patterns will be returned. |
| textextractor.no.text  | `false`       | Enable to stop `TextExtractor` from extracting any text at all. |
| track.anchors          | `true`        | Whether or not to add the anchor text (can be > 1) of (filtered) outgoing links with the key `anchors` to the Metadata of a tuple. |
| urlfilters.config.file | `urlfilters.json`| A JSON configuration file that defines URL filtering strategy. [Here is the default implementation](https://github.com/DigitalPebble/storm-crawler/blob/master/archetype/src/main/resources/archetype-resources/src/main/resources/urlfilters.json). Please also refer to [URLFilters](https://github.com/DigitalPebble/storm-crawler/wiki/URLFilters). **Note:** if you want to specify your own file you should give it a different name than `urlfilters.json`. For more information see [here](https://github.com/DigitalPebble/storm-crawler/issues/61) |

### Metadata

Options on how Storm Crawler should handle metadata tracking as well as
minimising metadata clashes

| key                    | default value | description                            |
|------------------------|---------------|----------------------------------------|
| metadata.persist       | -           | Which metadata to persist for a given document but *not* transfer to outlinks. Value is either a vector or a single valued String. `fetch.error.count` is always added |
| metadata.track.depth   | `true`        | Whether or not to track the depth of a crawled URL. This is a simple counter that is being tracked for outgoing links in the Metadata and incremented by `1` for every page that was crawled to find a specific link. This can be useful to let your Spout decide/sort which URLs to emit based on their depth. You could use this to influence the behavior of your recursive crawl (e.g. prefer pages with a low `depth` count). |
| metadata.track.path    | `true`        | Whether or not to track the URL path of outgoing links (all URLs that the crawler crawled to find this link) in the Metadata. The Metadata field name for this is `url.path`. It's a list of URLs that represent the crawl path (how did the crawler find this page). |
| metadata.transfer       | - | Which metadata to transfer to the outlinks and persist for a given document. Value is either a vector or a single valued String. |
| metadata.transfer.class | `com.digitalpebble.stormcrawler.util.MetadataTransfer` | Class to use for transfering metadata to outlinks. Must extend the class [MetadataTransfer](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/util/MetadataTransfer.java) |
| protocol.md.prefix     | -             | Prefix all metadata received by the remote server so that internal metadata with the same name (such as the remote IP address) is not overwritten. Further discussion in [issue 776](https://github.com/DigitalPebble/storm-crawler/issues/776) |
