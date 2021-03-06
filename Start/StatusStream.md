# Status Stream

The storm-crawler components rely on two Storm streams : the _**default**_ one and another one called _**status**_. 

The aim of the _status_ stream is to pass information about URLs to a persistence layer. Typically, a bespoke bolt will take the tuples coming from the _status_ stream and update the information about URLs in some sort of storage (e.g. ElasticSearch, HBase, etc...), which is then used by a Spout to send new URLs down the topology.

This is critical for building recursive crawls (i.e. you discover new URLs and not just process known ones). The _default_ stream is used for the URL being processed and is generally used at the end of the pipeline by an indexing bolt (which could also be ElasticSearch, HBase, etc...), regardless of whether the crawler is recursive or not.

Tuples are emitted on the _status_ stream by the parsing bolts for handling outlinks but also to notify that there has been a problem with a URL (e.g. unparsable content). It is also used by the fetching bolts to handle redirections, exceptions and unsuccessful fetch status (e.g. HTTP code 400).

A bolt which sends tuples on the _status_ stream declares its output in the following way \:
```
  declarer.declareStream(
                com.digitalpebble.storm.crawler.Constants.StatusStreamName,
                new Fields("url", "metadata", "status"));
```

as you can see for instance in [SimpleFetcherBolt](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/bolt/SimpleFetcherBolt.java#L149).

The [Status](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/persistence/Status.java) enum has the following values \:
* DISCOVERED \: outlinks found by the parsers or "seed" URLs emitted into the topology by one of the [spouts](https://stormcrawler.net/docs/api/com/digitalpebble/stormcrawler/spout/package-summary.html) or "injected" into the storage. The URLs can be already known in the storage.
* REDIRECTION \: set by the fetcher bolts.
* FETCH_ERROR \: set by the fetcher bolts.
* ERROR \: used by either the fetcher, parser or indexer bolts.
* FETCHED\: set by the StatusStreamBolt bolt (see below).

The difference between FETCH_ERROR and ERROR is that the former is possibly transient whereas the latter is terminal. The bolt which is in charge of updating the status (see below) can then decide when and whether to schedule a new fetch for a URL based on the status value.

The [DummyIndexer](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/indexing/DummyIndexer.java) is useful for notifying the storage layer that a URL has been successfully processed, i.e. fetched, parsed and anything else we want to do with the main content. It must be placed just before the StatusUpdaterBolt and sends a tuple for the URL on the status stream with a Status value of `fetched`. 

The class [AbstractStatusUpdaterBolt](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/persistence/AbstractStatusUpdaterBolt.java) can be extended to handle status updates for a specific backend. It has an internal cache of URLs with a `discovered` status so that they don't get added to the backend if they already exist, which is a simple but efficient optimisation. It also uses [DefaultScheduler](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/persistence/DefaultScheduler.java) to compute a next fetch date and calls MetadataTransfer to filter the metadata that will be stored in the backend.

In most cases, the extending classes will just need to implement the method `store(String URL, Status status, Metadata metadata,Date nextFetch)` and handle their own initialisation in `prepare()`. You can find an example of a class which extends in the [StatusUpdaterBolt](https://github.com/DigitalPebble/storm-crawler/blob/master/external/elasticsearch/src/main/java/com/digitalpebble/stormcrawler/elasticsearch/persistence/StatusUpdaterBolt.java) for Elasticsearch.