There are actually 2 different bolts for fetching the content of URLs.

* [SimpleFetcherBolt](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/storm/crawler/bolt/SimpleFetcherBolt.java)
* [FetcherBolt](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/storm/crawler/bolt/FetcherBolt.java)

Both declare the same output 

```
   declarer.declare(new Fields("url", "content", "metadata"));
        declarer.declareStream(
                com.digitalpebble.storm.crawler.Constants.StatusStreamName,
                new Fields("url", "metadata", "status"));
```

with the [status stream](statusStream) being used for handling redirections, restrictions by robots directives or fetch errors whereas the default stream gets the binary content returned by the server as well as the metadata to the following components (typically a parsing bolt).

Both use the same [protocol](Protocols) implementations and [URLFilters](URLFilters) to control the redirections.

The difference between these two implementations is that [FetcherBolt](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/storm/crawler/bolt/FetcherBolt.java) enforces politeness whereas [FetcherBolt](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/storm/crawler/bolt/FetcherBolt.java) doesn't.

The **FetcherBolt** has an internal set of queues where the incoming URLs are placed based on their hostname/domain/IP (see config 'fetcher.queue.mode') and a number of **FetchingThreads** (config `fetcher.threads.number` - 10 by default) which pull the URLS to fetch from the **FetchQueues**. When doing so, they make sure that a minimal amount of time (set with 'fetcher.server.delay' - default 1 sec) has passed since the previous URL was fetched from the same queue. This mechanism ensures that we can control the rate at which requests are sent to the servers. A **FetchQueue** can also be used by more than one **FetchingThread** at a time (in which case 'fetcher.server.min.delay' is used).

Incoming tuples spend very little time in the [execute](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/storm/crawler/bolt/FetcherBolt.java#L768) method of the **FetcherBolt** as they are put in the FetchQueues, which is why you'll find that the value of **Execute latency** in the Storm UI is pretty low. They get acked later on, after they've been fetched. The metric to watch for in the Storm UI is **Process latency**.

The **SimpleFetcherBolt** does not do any of this, hence its name. It just fetches incoming tuples in its execute method and does not enforce any politeness, nor does it do multi-threading. It is up to the user to declare multiple instances of the bolt in the Topology class and to manage how the URLs get distributed across the instances of SimpleFetcherBolt, often with the help of the [URLPartitionerBolt](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/storm/crawler/bolt/URLPartitionerBolt.java). The throttling of the fetching can also be done at the Spout level (config `topology.max.spout.pending`).


