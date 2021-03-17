# Welcome to the storm-crawler wiki!

## Getting Started
* [[Introduction]]
* [[Configuration]]: how to configure the storm-crawler
* [[Registering Metadata for Serialization]]: If your topology doesn't extend `ConfigurableTopology`, you will need to manually register storm-crawler's `Metadata` class for serialization in Storm.
* [Status Streams](StatusStream): Understanding how streams are used in Storm Crawler
* [[Debug with Eclipse]]

## Components
* Bolts
  * [[FetcherBolt(s)]]
  * [[IndexingBolts]]
  * [[JSoupParserBolt]]: parse HTML documents
  * [[SiteMapParserBolt]]: how to handle sitemaps
* Filters
  * [[ParseFilters]]: extract metadata from documents
  * [[URLFilters]]): how to filter or normalise outlinks
* Protocol
  * [[Protocols]]: Network protocols that are usable in storm-crawler

## Resources
* [[Presentations]]
* [[Powered By]]

## Sponsors
* [[Sponsors]]
