---
layout: ydndb-article
description: "Synchronization"
class: ydndb
title: "Synchronizing with a RESTful service"
introduction: "Automatically synchronize browser database with RESTful web service including AWS S3 and Google Cloud Storage."
article:
  written_on: 2013-05-18
  updated_on: 2014-05-18
  order: 3
collection: ydndb-sync
authors:
  - kyawtun
g_comments_href:
  http://dev.yathit.com/ydn-db/synchronization.html
---

{% wrap content %}


Synchronizing client database and backend RESTful server is fairly straight forwards. To archive efficient synchronization, the implementation is require more on server side than on client side. To facilitate client side data caching, HTTP protocol provide two (four methods) request header. Basically the state of the data is identified by either etag or updated date or both. If etag is used, a GET request include [If-None-Match](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.26) header of know etag value from the client side data. Server return 304 without response body if the data not changed. For modification requests, [If-Match](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.24) header is used to prevent overriding wrong version of a resource. The use case of modified date is similar.

A synchronization layer is required on top of CRUD methods (add, get, put, clear). The example [synchronization adapter](https://github.com/yathit/backbone-indexeddb-sync) is available on Github repository for Backbone framework on Google API backend. 

To notify changes in the database, storage instance dispatch StoreEvent and RecordEvent for changes in the database. The events are not dispatch by default because eventing implementation is not cheap. The desire store must installed to dispatch the events by setting `dispatchEvents` to `true`.

    var events_schema = {
      stores: [
        {
          name: 'store 1',
          keyPath: 'id',
          dispatchEvents: true,
          type: 'NUMERIC'}
      ]};
      
The events are listened as follow:
      
    db.addEventListener(['created', 'updated'], function(e) {
        console.log(e.name); // this will be either 'RecordEvent' or 'StoreEvent'
        console.log(e.getStoreName());
        console.log(e.type); // this will be one of 'created', 'updated' or 'deleted'
        // use getKey() and getValue() methods for RecordEvent
        // use getKeys() and getValues() methods for StoreEvent
    });
          
However to achieve efficiency in synchronization of collection of models, it is not straight forward because HTTP (and REST) do not said about conditional retrieval of collection of documents. A usual way is fetching with query parameter of last updated time in the client side. A demonstration of efficient retrieval using Google Data API (atom format) is available in the repository.

The following example use [Checkvist Open API](https://checkvist.com/auth/api), which follow REST principles for obtaining and updating
the data. Currently the API does not expose medata data on entity request header. For entity 'checklists', the resource URI and sample data for key
190078 is:

    https://beta.checkvist.com/checklists/190078.json</pre>
          <pre>{
      id: 190078
      name: "test sync app"
      public: false
      updated_at: "2013/06/08 22:57:09 +0000"
      user_count: 1
      user_updated_at: null
    }
    
[Store schema](/api/ydn/db/schema.html#sync) is declared as follow: 

    var schema = {
    stores: [{
      name: 'checklists',
      keyPath: 'id',
      Sync: {
        format: 'json',
        transport: service,
        Options: {
          baseUri: '/',
          delimiter: '/'
        }
      }
    }]
  };
  
The `Sync` attribute value specify synchronization information of the store. HTTP requests are made by HTTP transport
  service object specified by `transport` attribute in `Sync` options. A HTTP transport service is an object which has a method of `request`,
  which take argument object having fields of
  

* path - type: string: The URL to handle the request.
* method - type: string: The HTTP request method to use. Default is
    'GET'.
* params - type: Object: URL params in key-value pair form.
* headers - type: Object: Additional HTTP request headers.
* body - type: string: The HTTP request body (applies to PUT or POST).
* callback - type: Function: HTTP response object having the following
    fields described below.


Response object has the following fields


* url - type: string: The input URL.
* status - type: string: HTTP response status.
* contentType - type: string: HTTP response content type.
* reponseText - type: string: HTTP response text.
* response - type: Object: HTTP response.


It is similar to as defined in [Google Javascript client library](https://code.google.com/p/google-api-javascript-client/wiki/ReferenceDocs).
  
The following code snippet show how to make HTTP transport service:

    /**
     * Send request to checkvist server.
     * @link https://developers.google.com/api-client-library/javascript/reference/referencedocs#gapiclientrequest
     * @param {Object} args request arguments, compatible with Google javascript
     * client library.
     */
    CheckvistApp.prototype.request = function(args) {
      var path = args.path;
      var method = args.method || 'GET';
      var params = args.params || {};
      params['username'] = this.session.username;
      if (this.session.token) {
        params['token'] = this.session.token;
      }
      var host = 'beta.checkvist.com';
      var url = 'https://' + host + path + '.json';
      var query = [];
      for (var q in params) {
        query.push(q + '=' + encodeURIComponent(params[q]));
      }
      url += '?' + query.join('&amp;');
      var req = new XMLHttpRequest();
      req.open(method, url, false);
      // req.withCredentials = true;
      var me = this;
      req.onload = function(e) {
        var raw = {
          body: req.responseText,
          status: req.status,
          statusText: req.statusText,
          headers: {}
        };
    
        var header_lines = req.getAllResponseHeaders().split('\n');
        for (var i = 0; i &lt; header_lines.length; i++) {
          var idx = header_lines[i].indexOf(':');
          if (idx &gt; 0) {
            var name = header_lines[i].substr(0, idx).toLowerCase();
            var value = header_lines[i].substr(idx + 1).trim();
            raw.headers[name] = value;
          }
        }
    
        args.callback(JSON.parse(req.responseText), raw);
      };
      console.info('sending ' + url);
      req.send();
    };


A complete offline application can be found [Checkvist sync demo app](http://dev.yathit.com/demo/checkvist/checkvist-sync.html).
  
### Synchronizing with YDN-DB

YDN-DB have build in synchronization support for AWS S3 ([Amazon
  simple storage service](http://aws.amazon.com/documentation/s3/)), [ATOM
  syndicate format](http://www.ietf.org/rfc/rfc4287.txt), [OData](http://odata.org/), 
  [GData](https://developers.google.com/gdata/), GCS ([Google cloud storage](https://developers.google.com/storage/)).
  
    var schema = {
      stores: [
        {
          name: 'st',
          Sync: {
            format: 's3'
            transport: gapi.client
          }
        }
      ]
    };

{% endwrap %}        