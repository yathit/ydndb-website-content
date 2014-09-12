---
layout: ydndb-section
title: "Synchronization"
introduction: "Synchronization with backend server"
article:
  written_on: 2014-09-5
  updated_on: 2014-09-5
  order: 5
collection: ydndb-doc
id: ydndb-sync
authors:
  - kyawtun

---

{% wrap content %}

Data stored in client side storage mechanism is [ephemeral](https://developers.google.com/chrome/whitepapers/storage), although it may be possible to specified as persistent storage in the future through [quota management API](http://www.w3.org/TR/quota-api/). User agents are allowed to wipe out data anytime if deem necessary such as in case of low disk space or exceeding quota limit. For example mobile Safari browser will arbitrarily limit storage size to 50MB. Browser will not inform data lost to user or script. Critical data must be send back to the server.

Synchronization logic can be implemented by pre-database or in-database process.

In pre-database synchronization technique, database is used as secondary storage or caching. This technique required no additional functionality from database library. StoreEvent and RecordEvent dispatch from the storage instance may useful in this scenario.

In-database synchronization are handled by the library. Library usage is similar to non-synchornization usage. But, additional configuration and handler HTTP request are to be implemented. Before discussing synchronization process, we need to understand how the library handle cache invalidation and conflict resolution. These processes require metadata for each resource (record in this library, model in MVC architecture).

### Metadata

Metadata are specified in HTTP respond headers. For performance reason, several standardized URL format exist to retrieved only metadata for a specific resource or a list of resources (also know as collection resource). Metadata are store in client database along with the record value or in separate medata data store. Upon retrieval, these metadata are stripped from record value, but it may set to keep in record value as well through configuration. 

HTTP header name are case insensitive. This library always use lower case header name as metadata attribute name.

It should be noted that, for cross domain request, header name must be exposed setting [Access-Control-Expose-Headers](https://developer.mozilla.org/en-US/docs/HTTP/Access_control_CORS) in origin server.

[*etag*](http://www.w3.org/Protocols/rfc2616/rfc2616-sec3.html#sec3.11): Entity tag is used for cache invalidation and conditional retrieval and update.

[*last-modified*](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.29): Last modified time value is used for conflict resolution. In case of conflicting update in both client and server, an algorithm may automatically set last modified entity as winner. Additionally, if etag is not exposed, last modified time value is used for conditional retrieval and update. It should be note however that last modified time value is only accurate to second.

[*expires*](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.21): The Expires entity-header field gives the time after which the response is considered stale. If Expires header field is not expose, it is calculated from Cache-Control's [max-age](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9.3) value, if it presents.

*date*: Date refer to as current time value, which is not read from HTTP header, but set by using `Date.now()` at the time of storing metadata. Date value may be used to invalidate cache.

*key*: Resource key is URI path. Generally this is same as record key, but may have namespace prefix.

### Cache invalidation

Any database CRUD operation requires to invalidate the resource to origin server for freshness. There are two levels of invalidation for a read operation, opportunistic cache invalidation and conditional retrieval. Opportunistic cache invalidation use expires medata value for freshness. If resource entity is not expired, GET request is noted send to origin server for reading the entity. Immutable store also never send GET request if a record exist in the client database. Otherwise a conditional GET request is send either with `If-None-Match` etag or `If-Unmodified-Since` last-updated precondition. For write operation, `If-Match` etag or `If-None-Match` last-updated precondition is used.  

### Conflict resolution

All database write operation send the request with either with `ETag` or `Last-Updated` pre-condition. In case of failing precondition, the deferred object invoke fail callback with `ConflictEvent`, which encapsulate both client and server record value including metadata. If the Event object is not resolved in the fail callback, the event is resolve by default algorithm.           

{% endwrap %}        