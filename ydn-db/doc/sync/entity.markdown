---
layout: ydndb-article
description: "Synchronization using entity module"
class: ydndb
title: "Synchronizing with a web service"
introduction: "Automatically synchronize browser database with web service."
article:
  written_on: 2015-06-05
  updated_on: 2015-06-05
  order: 3
collection: ydndb-sync
authors:
  - kyawtun
  
---

{% wrap content %}


Synchronizing client database and backend RESTful server is fairly straight forwards. However efficient synchronization requires detail consideration. To facilitate client side data caching, HTTP protocol provide two (four methods) request header. Basically the state of the data is identified by either etag or updated date or both. If etag is used, a GET request include [If-None-Match](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.26) header of know etag value from the client side data. Server return 304 without response body if the data not changed. For modification requests, [If-Match](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.24) header is used to prevent overriding wrong version of a resource. The use case of modified date is similar. 

Optionally, for updating whole object store (collections) efficient, last modified timestamp is used as sync checkpoint. The client will request last known modified timestamp from the database and send request to server of all records later then the timestamp. 
 
YDN-DB `entity` provide conditional update for resource, last modified timestamp base updating and [write ahead logging](http://en.wikipedia.org/wiki/Write-ahead_logging) for offline write and restoring. However it is noted that YDN-DB only provide only necessary from client database. All these logic must implement in service provider, as describe below. 

The following example is base on [entity sync demo app](http://yathit.github.io/ydndb-demo/entity-sync/app.html) and its full source code is available in [github](https://github.com/yathit/ydndb-demo/tree/gh-pages/entity-sync). [Parse](https://parse.com) baas as used for backend service provider.

The object store managed by entity sync service must use out-of-line key with key generator (`autoIncrement = true`). Internally the key is constructed as composite key of resource id and validator token. When these values are not available, auto generated key is used. 
 
An sync history object store is required for entity sync service for storing write-ahead-log records. The following schema is used in the example. Multiple entities can be used in a database.  
 
    var schema = {
      stores: [{
        name: 'item',
        autoIncrement: true, // require by entity module, must not set keyPath.
        indexes: [{ // indexes are optional
          name: 'updatedAt',
          keyPath: 'updatedAt'
        }]
      }, {name: '_ydn_sync_history', // entity module requires this objectStore
        keyPath: 'sequence',
        autoIncrement: true,
        indexes: [
          {
            name: 'key',
            keyPath: ['entity', 'id']
          }
        ]
      }]
    };
    var db = new ydn.db.Storage('entity-sync-app', schema);   

The entity object, which is used for consuming the CRUD operation is created by `entity` method to the `Storage` instance providing service provider.

    var service = new ParseService('item', db);
    var Item = db.entity(service, 'item');
    
See [ParseService](https://github.com/yathit/ydndb-demo/blob/gh-pages/entity-sync/service.js) for detail. Service and Entity are created by name, so that multiple entities can be used.

Entity instance provide CRUD operations. All these operations will validate against with backend service. If backend service is not available, by returning 503 status code, the data are still updated to the database. In this case inconsistency are recorded in sync history object store. The application is responsible for resolving when backend is online. When backend and client database are in sync, there is no record in sync history object store.

Entity record is retrieved by `get` in this following `showRecord` method. 

    var showRecord = function(id) {
      var df = Item.get(id);
      df.progress(function(obj) {
        renderRecord(obj);
      });
      df.then(function(obj) {
        renderRecord(obj);
      }, function(e) {
        console.error(e);
      });
    };
    
If record is already available in client database, it is provided in the `progress` method. The same object is invoked again in promise resolve callback if backend service return `304` (Not Modified). Otherwise updated record from the backend service is provided in the promise resolve callback. `showRecord` can use `==` operator for checking modified status of the object.
  
To update whole object store, `update` method is used.  

    var updateRecord = function() {
      message('updating...');
      Item.update().then(function(cnt) {
        message(cnt + ' records updated from server.');
        if (cnt) {
          showRecent();
        }
      }, function(e) {
        console.error(e);
      });
      showRecent();
    };
    
`update` will invoke `list` method to service provider.  
  
    RestService.prototype.list_ = function(cb, name, token) {
      Rest.list(function(arr) {
        var ids = arr.map(function(obj) {
          return obj.objectId;
        });
        var tokens = arr.map(function(obj) {
          return obj.updatedAt;
        });
        cb(200, arr, ids, tokens);
      }, token);
    };
    
    
    /**
     * @override
     */
    RestService.prototype.list = function(cb, name, token) {
      if (token) {
        RestService.prototype.list_(cb, name, token);
      } else {
        this.db.values(this.name, 'updatedAt', null, 1, 0, true).always(function(obj) {
          var token = obj[0] ? obj[0].updatedAt : null;
          RestService.prototype.list_(cb, name, token);
        });
      }
    };
    
Since the serivce is using last modified data are incremental updating strategy, `list` method retrieve the last modified record from the client database using `updatedAt` index as described in above code snippet. It is noticed that YDN-DB entity module itself does not assume any updating strategy is used. Other updating stragegy such as incresing or descresing key can be used for immutable data in S3 service.    
 
 

{% endwrap %}        