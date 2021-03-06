The URL filters can be used to both remove or modify incoming URLs (unlike Nutch where these functionalities are separated between URLFilters and URLNormalizers). This is used generally within a parsing bolt to normalise and filter outgoing URLs, but is also called within the FetcherBolt to handle redirections.

URLFilters need to implement the interface [URLFilter](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/filtering/URLFilter.java) which defines a single method : 

```
public String filter (URL sourceUrl, Metadata sourceMetadata,
            String urlToFilter);
```

and inherits a default one from [Configurable](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/util/Configurable.java)

```
public void configure(Map stormConf, JsonNode jsonNode);
```

The configuration is done via a JSON file which is loaded by the wrapper class [URLFilters](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/filtering/URLFilters.java). The URLFilter instances can be used directly but it is easier to use the class URLFilters instead. Some filter implementations can also be configured with the [standard configuration mechanism](https://github.com/DigitalPebble/storm-crawler/wiki/Configuration).

Here is an example of a [JSON configuration file](https://github.com/DigitalPebble/storm-crawler/blob/master/archetype/src/main/resources/archetype-resources/src/main/resources/urlfilters.json).

The JSON configuration allows to load several instances of the same filtering class with different parameters and can handle complex configuration objects since it makes no assumptions about the content of the field 'param'. The URLFilters are executed in the order in which they are defined in the JSON file.

The following URL filters are provided as part of the core code.

**Basic**
The [BasicURLFilter](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/filtering/basic/BasicURLFilter.java) filters based on the length of the URL and the repetition of path elements. 

The [BasicURLNormalizer](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/filtering/basic/BasicURLNormalizer.java) removes the anchor part of URLs based on the value of the parameter `removeAnchorPart`. It also removed query elements based on the configuration and whether their value corresponds to a 32-bit hash.

**FastURLFilter**
The [FastURLFilter](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/filtering/regex/FastURLFilter.java
) is based on regex patterns and organised by scope [host | domain | metadata | global]. For a given URL, the scopes are tried in the order given above and the URL is kept or removed based on the first matching rule. The default policy is to accept a URL if no match is found.

The resource file is in JSON and looks like so:

```
  [{
          "scope": "GLOBAL",
          "patterns": [
              "DenyPathQuery \\.jpg"
          ]
      },
      {
          "scope": "domain:stormcrawler.net",
          "patterns": [
              "AllowPath /digitalpebble/",
              "DenyPath .+"
          ]
      },
      {
          "scope": "metadata:key=value",
          "patterns": [
             "DenyPath .+"
          ]
      }
  ]
```

_DenyPathQuery_ indicates that the pattern should be applied on the path URL path and the query element, whereas _DenyPath_ means the path alone.

**Host**
The [HostURLFilter](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/filtering/host/HostURLFilter.java) filters URLs based on whether they belong to the same host or domain name as the source URL. This is configured with the parameter `ignoreOutsideDomain` and `ignoreOutsideHost`. The latter takes precedence over the former.

**MaxDepth**
The [MaxDepthFilter](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/filtering/depth/MaxDepthFilter.java) is configured with the parameter `max Depth` and requires `metadata.track.depth` to be set to true in the [[Configuration]]. This removes outlinks found too far from the seed URL and is a way of controling the expansion of the crawl.

If the filter is configured with a value of 0, then all outlinks will be removed, regardless of whether the depth is being tracked.

The max depth can also be set on a per seed basis, using the key/value `max.depth`, which is automatically transfered to the outlinks if `metadata.track.depth` is set to true.

**Metadata**
The [MetadataFilter](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/filtering/metadata/MetadataFilter.java) filters URLs based on metadata in the source document.

**RegexURLFilter**
The [RegexURLFilter](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/filtering/regex/RegexURLFilter.java) uses a [configuration file](https://github.com/DigitalPebble/storm-crawler/blob/master/archetype/src/main/resources/archetype-resources/src/main/resources/default-regex-filters.txt) or a JSON ArrayNode containing regular expressions to determine whether a URL should be kept or not. The most specific rule must be placed first as a URL is kept or removed based on the first matching rule.

<br/>
RegexURLFilter config sample in JSON format:<br/>
```
{
    "urlFilters": [
        "-^(file|ftp|mailto):",
        "+."
    ]
}
```

**RegexURLNormalizer**
The [RegexURLNormalizer](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/filtering/regex/RegexURLNormalizer.java) uses a [configuration file](https://github.com/DigitalPebble/storm-crawler/blob/master/archetype/src/main/resources/archetype-resources/src/main/resources/default-regex-normalizers.xml) or a JSON ArrayNode containing regular expressions and replacements to normalize URLs.
<br/>
RegexURLNormalizer config sample in JSON format:<br/>
```
{
    "urlNormalizers": [
        {
            "pattern": "#.*?(\\?|&amp;|$)",
            "substitution": "$1"
        },
        {
            "pattern": "\\?&amp;",
            "substitution": "\\?"
        }
    ]
}
```

**RobotsFilter**
The [RobotsFilter](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/filtering/robots/RobotsFilter.java) discards URLs based on the robots.txt directives. This is meant to be used on small, limited crawls where the number of hosts is finite. Using this on a larger or open crawl would have a negative impact on performance as the filter would try to retrieve the robots.txt files for any host found.

**SitemapFilter**
The [SitemapFilter](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/filtering/sitemap/SitemapFilter.java) discards the outlinks of URLs which are not sitemaps when sitemaps have been found.
