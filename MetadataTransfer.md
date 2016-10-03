The class [MetadataTransfer](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/util/MetadataTransfer.java) is  an important part of the framework and is used in key parts of a pipeline.

* Fetching
* Parsing
* Updating bolts

An instance (or extension) of MetadataTransfer gets created and configured with the method `public static MetadataTransfer getInstance(Map<String, Object> conf)` which takes as parameter with the standard Storm [[Configuration]]. 

A **MetadataTransfer** instance has mainly two methods, both returning Metadata objects \:

* `getMetaForOutlink(String targetURL, String sourceURL,
            Metadata parentMD)`
*  `filter(Metadata metadata)`  

The former is used when creating [Outlinks](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/parse/Outlink.java) i.e. in the parsing bolts but also for handling redirections in the [[FetcherBolt(s)]].

The latter is used by extensions of the [AbstractStatusUpdaterBolt](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/persistence/AbstractStatusUpdaterBolt.java) class to determine which **Metadata** should be persisted.

The behaviour of the default MetadataTransfer class is driven by configuration only. It has the following options. 

* `metadata.transfer` list of metadata key values to filter or transfer to the outlinks. See [https://github.com/DigitalPebble/storm-crawler/blob/master/core/crawler-conf.yaml#L11]
* `metadata.track.path` whether to track the url path or not. Boolean value, true by default.
* `metadata.track.depth` whether to track the depth from seed. Boolean value, true by default.

Note that the method `getMetaForOutlink` calls `filter` to determine what to key values to keep.
