* [PopSugar  - JSOUP parser](https://github.com/PopSugar/storm-crawler-extensions/tree/master/jsoup-parser)

Similar to the ParserBolt. The difference being that it uses JSoup to do the HTML Parsing instead of Tika.

* [PopSugar  - microdata parser](https://github.com/PopSugar/storm-crawler-extensions/tree/master/microdata-parser)

A ParseFilter to be plugged in with the ParserBolt (or the JSoupParser above). It extracts all the microdata stored in the page and adds it to the metadata map using a key-dotted path.