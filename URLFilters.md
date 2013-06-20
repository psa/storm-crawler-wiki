The URL filters can be used to both remove or modify incoming URLS (unlike Nutch where these functionalities are separated between URLFilters and URLNormalizers).

URLFilters need to implement the interface URLFilter which has two methods : 

_public String filter (String URL);_

_public void configure(JsonNode jsonNode);_

The configuration is done via a JSON file which is loaded by the wrapper class URLFilters. The URLFilter instances can be used directly but it is easier to use the class URLFilters instead.

Here is an example of a [JSON configuration file](https://github.com/DigitalPebble/storm-crawler/blob/master/src/main/resources/urlfilters.json).

The JSON configuration allows to load several instances of the same filtering class with different parameters,  allows complex configuration objects since it makes no assumptions about the content of the field 'param'. The URLFilters are executed in the order in which they are defined in the JSON file.

