The following network protocols are implemented in storm-crawler:

# File
* [FileProtocol](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/protocol/file/FileProtocol.java)

# HTTP/S

See [[HTTPProtocol]] for the effect of metadata content on protocol behaviour.

* [HttpClient](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/protocol/httpclient/HttpProtocol.java)
* [Selenium](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/protocol/selenium/SeleniumProtocol.java)
* [OKHttp](https://github.com/DigitalPebble/storm-crawler/blob/master/core/src/main/java/com/digitalpebble/stormcrawler/protocol/okhttp/HttpProtocol.java)

## Feature grid

| Features             | HTTPClient | OKhttp | Selenium |
|----------------------|:----------:|:------:|:--------:|
| Basic authentication |      [Y](https://github.com/DigitalPebble/storm-crawler/pull/589)     |    N   |     N    |
| proxy (w. credentials?) |       Y / Y     |  Y / N      |      ?    |
| interruptible / trimmable [#463](https://github.com/DigitalPebble/storm-crawler/issues/463)|    N / Y       |   Y / Y    |    Y / N      |
| cookies                   |     Y       |   [Y](https://github.com/DigitalPebble/storm-crawler/issues/632)     |    N      |
| response headers                   |     Y       |   Y     |    N      |
| trust all certificates                  |     N       |   [Y](https://github.com/DigitalPebble/storm-crawler/issues/615)      |    N      |
| HEAD method [#485](https://github.com/DigitalPebble/storm-crawler/issues/485)|     Y       |   N     |    N      |
| verbatim response header |  [Y](https://github.com/DigitalPebble/storm-crawler/issues/317)     |   [Y](https://github.com/DigitalPebble/storm-crawler/issues/506)    |    N      |
| verbatim request header |  [N](https://github.com/DigitalPebble/storm-crawler/issues/317)     |    [Y](https://github.com/DigitalPebble/storm-crawler/issues/506)    |    N      |
| navigation and javascript | N | N | Y |