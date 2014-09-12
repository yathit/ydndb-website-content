---
layout: ydndb-article
description: "Synchronization"
class: ydndb
title: "Synchronizing with AWS S3 like backend"
introduction: "Automatically synchronize browser database with RESTful web service including AWS S3 and Google Cloud Storage."
article:
  written_on: 2013-05-18
  updated_on: 2014-05-18
  order: 4
collection: ydndb-sync
authors:
  - kyawtun
g_comments_href:
  http://dev.yathit.com/ydn-db/synchronization.html
---

{% wrap content %}

Synchronizing with RESTful service is pretty straight forward. The following example use [Checkvist Open API](https://checkvist.com/auth/api), which follow REST principles for obtaining and updating
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
  
<h3>Synchronizing with YDN-DB</h3>
<p>YDN-DB have build in synchronization support for AWS S3 (<a href="http://aws.amazon.com/documentation/s3/">Amazon
  simple storage service</a>), <a href="http://www.ietf.org/rfc/rfc4287.txt">ATOM
  syndicate format</a>, <a href="http://odata.org/">OData</a>, <a href="https://developers.google.com/gdata/">
  GData</a>, GCS (<a href="https://developers.google.com/storage/">Google
  cloud storage</a>).</p>
      <pre>var schema = {
  stores: [
    {
      name: 'st',
      Sync: {
        format: 's3'
        transport: gapi.client
      }
    }
  ]
};</pre>

{% endwrap %}        