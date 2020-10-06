The following network protocols are implemented in StormCrawler:

# File
* [FileProtocol](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/protocol/file/FileProtocol.java)

# HTTP/S

See [[HTTPProtocol]] for the effect of metadata content on protocol behaviour.

To change the implementation, add the following lines to your _crawler-conf.yaml_

```
  http.protocol.implementation: "com.digitalpebble.stormcrawler.protocol.okhttp.HttpProtocol"
  https.protocol.implementation: "com.digitalpebble.stormcrawler.protocol.okhttp.HttpProtocol"
```

* [HttpClient](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/protocol/httpclient/HttpProtocol.java)
* [Selenium](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/protocol/selenium/SeleniumProtocol.java)
* [OKHttp](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/protocol/okhttp/HttpProtocol.java)

## Feature grid

| Features             | HTTPClient | OKhttp | Selenium |
|----------------------|:----------:|:------:|:--------:|
| Basic authentication |      [Y](https://github.com/DigitalPebble/storm-crawler/pull/589)     |    [Y](https://github.com/DigitalPebble/storm-crawler/issues/792)  |     N    |
| proxy (w. credentials?) |       Y / Y     |  Y / [Y](https://github.com/DigitalPebble/storm-crawler/issues/751)      |      ?    |
| interruptible / trimmable [#463](https://github.com/DigitalPebble/storm-crawler/issues/463)|    N / Y       |   Y / Y    |    Y / N      |
| cookies                   |     Y       |   [Y](https://github.com/DigitalPebble/storm-crawler/issues/632)     |    N      |
| response headers                   |     Y       |   Y     |    N      |
| trust all certificates                  |     N       |   [Y](https://github.com/DigitalPebble/storm-crawler/issues/615)      |    N      |
| HEAD method [#485](https://github.com/DigitalPebble/storm-crawler/issues/485)|     Y       |   N     |    N      |
| POST method [#641](https://github.com/DigitalPebble/storm-crawler/issues/641)|     N       |   Y     |    N      |
| verbatim response header |  [Y](https://github.com/DigitalPebble/storm-crawler/issues/317)     |   [Y](https://github.com/DigitalPebble/storm-crawler/issues/506)    |    N      |
| verbatim request header |  [N](https://github.com/DigitalPebble/storm-crawler/issues/317)     |    [Y](https://github.com/DigitalPebble/storm-crawler/issues/506)    |    N      |
| navigation and javascript | N | N | Y |
| HTTP/2               | N      | Y  |  (Y)  |

## HTTP/2

- the [OKHttp](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/protocol/okhttp/HttpProtocol.java) protocol supports [HTTP/2](https://en.wikipedia.org/wiki/HTTP/2) if the JDK includes [ALPN](https://en.wikipedia.org/wiki/Application-Layer_Protocol_Negotiation) (Java 9 and [Java 8 builds starting early/mid 2020](https://mail.openjdk.java.net/pipermail/jdk8u-dev/2020-January/011042.html)).
- [HttpClient](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/protocol/httpclient/HttpProtocol.java) does not yet support HTTP/2
- [Selenium](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/protocol/selenium/SeleniumProtocol.java): whether HTTP/2 is used or not depends on the used driver

Since [#829](https://github.com/DigitalPebble/storm-crawler/pull/829) the HTTP protocol version used is configurable via `http.protocol.versions` (see also comments in [crawler-default.yaml](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/resources/crawler-default.yaml). Eg., to force that only HTTP/1.1 is used:
```
http.protocol.versions:
- "http/1.1"
```

