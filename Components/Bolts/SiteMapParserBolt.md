Storm-crawler can handle sitemap files thanks to the **SiteMapParserBolt**. This bolt should be placed before the standard **ParserBolt** in the topology, as illustrated in [CrawlTopology](https://github.com/DigitalPebble/storm-crawler/blob/master/archetype/src/main/resources/archetype-resources/src/main/java/CrawlTopology.java).

The reason for this is that the **SiteMapParserBolt** acts as a filter: it passes on any incoming tuples to the default stream so that it gets processed by the **ParserBolt**, unless the tuple contains `isSitemap=true` in its metadata, in which case the **SiteMapParserBolt** will parse it itself. Any outlinks found in the sitemap files are then emitted on the [[StatusStream]].

The **SiteMapParserBolt** applies any configured [[ParseFilters]] to the documents it parses and just like its equivalent for HTML pages, it uses [[MetadataTransfer]] to populate the Metadata objects for the Outlinks it finds.
