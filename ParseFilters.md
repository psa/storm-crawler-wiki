**ParseFilters** are called from parsing bolts such as  [JSoupParserBolt](https://github.com/DigitalPebble/storm-crawler/wiki/JSoupParserBolt) and [SiteMapParserBolt](https://github.com/DigitalPebble/storm-crawler/wiki/SiteMapParserBolt) to extract data from web pages. The data extracted are stored in the Metadata object. ParseFilters can also modify the **Outlinks** and in that sense act as [[URLFilters]].

ParseFilters need to implement the interface [ParseFilter](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/storm/crawler/parse/ParseFilter.java) which has three methods :

```
public void filter(String URL, byte[] content, DocumentFragment doc,
            Metadata metadata, List<Outlink> outlinks);

public void configure(Map stormConf, JsonNode filterParams);

public boolean needsDOM();
```
The `filter` method is where the extraction takes place.

The `needsDOM` method simply indicates whether a given ParseFilter instance needs the DOM structure. If no ParseFilters need it, the parsing bolt will not generate the DOM which can slightly improve the performance.

The `configure` method takes as input a JSON file which is loaded by the wrapper class ParseFilters. The config map from Storm can also be used to configure the filters, as explained in [[Configuration]].

Here is the default [JSON configuration file](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/resources/parsefilters.json) for ParseFilters.

The JSON configuration allows to load several instances of the same filtering class with different parameters, allows complex configuration objects since it makes no assumptions about the content of the field 'param'. The ParseFilters are executed in the order in which they are defined in the JSON file.

The following ParseFilters are provided as part of the core code.

**XPathFilter**
The [XPathFilter](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/storm/crawler/parse/filter/XPathFilter.java) allows to define XPath expressions to extract data from the page and store them in the Metadata object. 

**DebugParseFilter**
The [DebugParseFilter](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/storm/crawler/parse/filter/DebugParseFilter.java) dumps a XML representation of the DOM structure to a temporary file.


