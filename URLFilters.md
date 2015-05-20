The URL filters can be used to both remove or modify incoming URLS (unlike Nutch where these functionalities are separated between URLFilters and URLNormalizers). This is used generally within a parsing bolt to normalise and filter outgoing URLs, but is also called within the FetcherBolt to handle redirections.

URLFilters need to implement the interface [URLFilter](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/storm/crawler/filtering/URLFilter.java) which has two methods : 

_public String filter (URL sourceUrl, Metadata sourceMetadata,
            String urlToFilter);_

_public void configure(Map stormConf, JsonNode jsonNode);_

The configuration is done via a JSON file which is loaded by the wrapper class [URLFilters](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/storm/crawler/filtering/URLFilters.java). The URLFilter instances can be used directly but it is easier to use the class URLFilters instead. The config map from Storm can also be used to configure the filters.

Here is an example of a [JSON configuration file](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/resources/urlfilters.json).

The JSON configuration allows to load several instances of the same filtering class with different parameters,  allows complex configuration objects since it makes no assumptions about the content of the field 'param'. The URLFilters are executed in the order in which they are defined in the JSON file.

The following URL filters are provided as part of the core code.

**Basic**
The [BasicURLFilter](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/storm/crawler/filtering/basic/BasicURLNormalizer.java) removes the anchor part of URLs based on the value of the parameter `removeAnchorPart`.

**MaxDepth**
The [MaxDepthFilter]https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/storm/crawler/filtering/depth/MaxDepthFilter.java) is configured with the parameter `max Depth` and requires `metadata.track.depth` to be set to true in the [[Configuration]]. This removes outlinks found too far from the seed URL and is a way of controling the expansion of the crawl.

**Host**
The [HostURLFilter](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/storm/crawler/filtering/host/HostURLFilter.java) filters URLs based on whether they belong to the same host or domain name as the source URL. This is configured with the parameter `ignoreOutsideDomain` and `ignoreOutsideHost`. The latter takes precedence over the former.

**RegexURLFilter**
The [RegexURLFilter](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/storm/crawler/filtering/regex/RegexURLFilter.java) uses a [configuration file](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/resources/default-regex-filters.txt) or a JSON ArrayNode containing regular expressions to determine whether a URL should be kept or not.
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
The [RegexURLNormalizer](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/storm/crawler/filtering/regex/RegexURLNormalizer.java) uses a [configuration file](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/resources/default-regex-normalizers.xml) or a JSON ArrayNode containing regular expressions and replacements to normalize URLs.
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