**ParseFilters** are called from parsing bolts such as  [JSoupParserBolt](https://github.com/DigitalPebble/storm-crawler/wiki/JSoupParserBolt) and [SiteMapParserBolt](https://github.com/DigitalPebble/storm-crawler/wiki/SiteMapParserBolt) to extract data from web pages. The data extracted are stored in the Metadata object. ParseFilters can also modify the **Outlinks** and in that sense act as [[URLFilters]].

ParseFilters need to implement the interface [ParseFilter](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/parse/ParseFilter.java) which has three methods :

```
public void filter(String URL, byte[] content, DocumentFragment doc,
            ParseResult parse);

public void configure(Map stormConf, JsonNode filterParams);

public boolean needsDOM();
```
The `filter` method is where the extraction takes place. [ParseResult](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/parse/ParseResult.java) objects contain the outlinks extracted from the document as well as a Map of String [ParseData](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/storm/crawler/parse/ParseData.java) where the String is the URL of a subdocument obtained from the main one (or the main one itself). The ParseData objects contain Metadata, binary content and text for the subdocuments. This is useful for indexing subdocuments independently from the main document. 

The `needsDOM` method simply indicates whether a given ParseFilter instance needs the DOM structure. If no ParseFilters need it, the parsing bolt will not generate the DOM which can slightly improve the performance.

The `configure` method takes as input a JSON file which is loaded by the wrapper class ParseFilters. The config map from Storm can also be used to configure the filters, as explained in [[Configuration]].

Here is the default [JSON configuration file](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/resources/parsefilters.json) for ParseFilters.

The JSON configuration allows to load several instances of the same filtering class with different parameters, allows complex configuration objects since it makes no assumptions about the content of the field 'param'. The ParseFilters are executed in the order in which they are defined in the JSON file.

The following ParseFilters are provided as part of the core code.

**XPathFilter**
The [XPathFilter](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/parse/filter/XPathFilter.java) allows to define XPath expressions to extract data from the page and store them in the Metadata object. 

**DebugParseFilter**
The [DebugParseFilter](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/parse/filter/DebugParseFilter.java) dumps a XML representation of the DOM structure to a temporary file.

**ContentFilter**
The [ContentFilter](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/parse/filter/ContentFilter.java) allows to restrict the text of a document to the text covered by a Xpath expression.

**LinkParseFilter**
The [LinkParseFilter](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/parse/filter/LinkParseFilter.java) can be used to extract outlinks from documents using Xpath expressions defined in the config.

**DomainParseFilter**
The [DomainParseFilter](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/parse/filter/DomainParseFilter.java) stores the domain or host name in the metadata for indexing later on.

**MD5SignatureParseFilter**
The [MD5SignatureParseFilter](
https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/parse/filter/MD5SignatureParseFilter.java) generates a MD5 signature of a document based on the binary content, text or URL (as a last resort). Can be used in combination with the *ContentFilter* above so that the text used for the signature excludes any boilerplate.
