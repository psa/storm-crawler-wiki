# Status Stream

The storm-crawler components rely on two Storm streams : the _default_ one and another one called _status_. 

The aim of the _status_ stream is to pass information about URLs to a persistence layer. Typically, a bespoke bolt will take the tuples coming from the _status_ stream and update the information about URLs in some sort of storage (e.g. ElasticSearch, Hbase, etc...), which is then used by a Spout to send new URLs down the topology.

This is relevant for recursive crawls only. If the crawler you are building with storm-crawler is non-recursive (i.e. you don't discover new URLs but only process known ones) then you probably don't need to use the status stream.

TODO what is it used for? newly discovered URLs, exceptions, etc...

TODO : status, fetchDate etc...

The _default_ stream is used for the URL being processed and is generally used at the end of the pipeline by an indexing bolt (which could also be ElasticSearch, Hbase, etc...), regardless of whether the crawler is recursive or not.
