The purpose of crawlers is often to index web pages to make them searchable. The project contains resources for indexing with popular search solutions such as [Apache SOLR](https://github.com/DigitalPebble/storm-crawler/blob/master/external/solr/src/main/java/com/digitalpebble/stormcrawler/solr/bolt/IndexerBolt.java),  [Elasticsearch](https://github.com/DigitalPebble/storm-crawler/blob/master/external/elasticsearch/src/main/java/com/digitalpebble/stormcrawler/elasticsearch/bolt/IndexerBolt.java) or [AWS Cloudsearch](https://github.com/DigitalPebble/storm-crawler/blob/master/external/aws/src/main/java/com/digitalpebble/stormcrawler/aws/bolt/CloudSearchIndexerBolt.java), which all extend the class [AbstractIndexerBolt](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/indexing/AbstractIndexerBolt.java).

The core module also contains a [simple indexer](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/indexing/StdOutIndexer.java) which dumps the documents into the standard output - which can be useful for debugging as well as a [DummyIndexer](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/indexing/DummyIndexer.java).

The basic functionalities of filtering a document to index, mapping the metadata (which determines which metadata to keep for indexing and under what field name) or using the canonical tag (if any) are handled by the abstract class, which means that the implementations can focus on the communication with the indexing APIs.

The indexing is often the penultimate component in a pipeline and takes the output of a Parsing bolt on the standard stream. The output of the indexing bolts is on the _status_ stream \:

```
    public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declareStream(
                com.digitalpebble.stormcrawler.Constants.StatusStreamName,
                new Fields("url", "metadata", "status"));
    }
```

The [DummyIndexer](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/indexing/DummyIndexer.java) is used for cases where no actual indexing is required and it simply generates a tuple on the _status_ stream so that any StatusUpdater bolt knows that the URL was processed successfully and can update its status and scheduling in the corresponding backend. 