The URL filters can be used to both remove or modify incoming URLS (unlike Nutch where these functionalities are separated between URLFilters and URLNormalizers).

URLFilters need to implement the interface URLFilter which has two methods : 

_public String filter (URL sourceUrl, Metadata sourceMetadata,
            String urlToFilter);_

_public void configure(Map stormConf, JsonNode jsonNode);_

The configuration is done via a JSON file which is loaded by the wrapper class URLFilters. The URLFilter instances can be used directly but it is easier to use the class URLFilters instead. The config map from Storm can also be used.

Here is an example of a [JSON configuration file](https://github.com/DigitalPebble/storm-crawler/blob/master/src/main/resources/urlfilters.json).

The JSON configuration allows to load several instances of the same filtering class with different parameters,  allows complex configuration objects since it makes no assumptions about the content of the field 'param'. The URLFilters are executed in the order in which they are defined in the JSON file.

The following URL filters are provided as part of the core code.

**Basic**
The [BasicURLFilter](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/storm/crawler/filtering/basic/BasicURLFilter.java) removes the anchor part of URLs based on the value of the parameter `removeAnchorPart`.

**Host**
The [HostURLFilter](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/storm/crawler/filtering/host/HostURLFilter.java) filters URLs based on whether they belong to the same host or domain name as the source URL. This is configured with the parameter `ignoreOutsideDomain` and `ignoreOutsideHost`. The latter takes precedence over the former.

**RegexURLFilter**
The [RegexURLFilter](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/storm/crawler/filtering/regex/RegexURLFilter.java) uses a [configuration file](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/resources/default-regex-filters.txt) containing regular expressions to determine whether a URL should be kept or not.

**RegexURLNormalizer**
The [RegexURLNormalizer](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/storm/crawler/filtering/regex/RegexURLNormalizer.java) uses a [configuration file](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/resources/default-regex-normalizers.xml) containing regular expressions and replacements to normalize URLs.

