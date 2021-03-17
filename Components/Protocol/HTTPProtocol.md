## Metadata-dependent Behavior

The `metadata` argument to [HTTPProtocol.getProtocolOutput()](https://stormcrawler.net/docs/api/com/digitalpebble/stormcrawler/protocol/Protocol.html#getProtocolOutput-java.lang.String-com.digitalpebble.stormcrawler.Metadata-) can affect the behavior of the protocol. The following metadata keys are detected by `HTTPProtocol` implementations and utilized in performing the request:

* `last-modified`: If this key is present in `metadata`, the protocol will use the metadata value as the date for the `If-Modified-Since` header field of the HTTP request. If the key is not present, the `If-Modified-Since` field won't be added to the request header.

* `protocol.etag`: If this key is present in `metadata`, the protocol will use the metadata value as the ETag for the `If-None-Match` header field of the HTTP request. If the key is not present, the `If-None-Match` field won't be added to the request header.

* `http.accept`: If this key is present in `metadata`, the protocol will use the value to override the value for the `Accept` header field of the HTTP request. If the key is not present, the `http.accept` global configuration value is used instead. (Available in v1.11+)

* `http.accept.language`: If this key is present in `metadata`, the protocol will use the value to override the value for the `Accept-Language` header field of the HTTP request. If the key is not present, the `http.accept.language` global configuration value is used instead. (Available in v1.11+)

* `protocol.set-cookie`: If this key is present in `metadata` and `http.use.cookies` is true, the protocol will sent cookies stored from the response this page was linked to given the cookie is applicable to the domain of the link.

* `http.method.head`: If this key is present in `metadata`, the protocol sends a HEAD request. (Available in v1.12+ only for httpclient, see [#485](https://github.com/DigitalPebble/storm-crawler/issues/485))

* `http.post.json`: If this key is present in `metadata`, the protocol sends a POST request. (Available in v1.12+ only for okhttp, see [#641](https://github.com/DigitalPebble/storm-crawler/issues/641))


Notes:
- metadata values starting with `protocol.` may start with a different prefix instead, see `protocol.md.prefix` and [#776](https://github.com/DigitalPebble/storm-crawler/issues/776)
- metadata used for requests needs to be persisted, e.g.
  ```yaml
  metadata.persist:
   - last-modified
   - protocol.etag
   - protocol.set-cookie
   - ...
  ```
- cookies need to be transferred to outlinks by setting
  ```yaml
  metadata.transfer:
   - set-cookie
  ```
