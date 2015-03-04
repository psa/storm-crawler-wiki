## Metadata-dependent Behavior

The `knownMetadata` argument to `HTTPProtocol.getProtocolOutput()` can affect the behavior of the protocol. The following metadata keys are detected by `HTTPProtocol` and utilized in performing the request:

* `cachedLastModified`: If this key is present in `knownMetadata`, the protocol will use the metadata value as the date for the `If-Modified-Since` header field of the HTTP request. If the key is not present, the `If-Modified-Since` field won't be added to the request header.

* `cachedEtag`: If this key is present in `knownMetadata`, the protocol will use the metadata value as the ETag for the `If-None-Match` header field of the HTTP request. If the key is not present, the `If-None-Match` field won't be added to the request header.