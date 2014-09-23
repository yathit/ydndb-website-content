---
layout: ydndb-article
description: "NoSQL query"
class: ydndb
title: "AngularJS app on Sync"
introduction: "This Angular app demonstrates how to use synchronization with Google Cloud Storage service."
article:
  written_on: 2013-04-01
  updated_on: 2014-04-28
  order: 4
collection: ydndb-example
authors:
  - kyawtun
---

{% wrap content %}
 
### Feature matrix

This Angular app demonstrates how to use synchronization. [Angular](https://angularjs.org/) is one of the popular framework for developing modern web app.

Google Cloud Storage (GCS) backend server is used to synchronized data with client side database. GCS backend is essentially a RESTFul API. Only new records are delivered to the client by using monotonic increasing key.

## An angular service for GCS backend

Angular JS application has four primary modules, 1) Controller 2) Directive 3) Filter and 4) Services. Each of them are essentially a singleton instance providing function that angular understand.

YDN-DB database instance belong to service module, the lowest food chain of the four angular modules. Since YDN-DB storage instance provide ORM like functions, we do not need any additional wrapper function here. We define schema and instantiate a new storage instance. If localStorage is used, we will need to serialize and deserialize during write and write. Web SQL and IndexedDB will require more function for creating transaction and making request.

    angular.module('myApp.services', [])
        .factory('database', function() {
          var schema = {}; // detail later
          return new ydn.db.Storage('feature-matrix', schema);
        });
        
There is on thing to note. Most of the web app examples, that you might found are synchronous, meaning that database service return query result. However YDN-DB always return asynchronously in the form of JQuery compatible promise. AngularJS also use [JQuery compatible promise](http://api.jquery.com/promise/). Here, compatible means you can directly chain the promise. However AngularJS promise, $q, has optimization twist. $q deferred resolution synchronize with AngularJS render event cycle so that DOM manipulation are orchestrated to reduce browser point refresh. However YDN-DB deferred, nor JQuery deferred, aware of them. Teh application logic has to to pump event up by invoking [$scope.$apply()](http://docs.angularjs.org/api/ng.$rootScope.Scope#methods_$apply) for each deferred resolution. We will see this in controller, where the database service is consumed.  
      
To enable synchronization between client database and backend server, [Sync attribute](http://dev.yathit.com/api-reference/ydn-db/schema.html#sync) is added into the database store schema by specifying backend service type and name. Basic format will be 'rest', representing RESTful backend server. Here Google Cloud Storage backend service, first class citizen in YDN-DB, is used by specifying gcs format, which is essentially equals to s3 format. The only require attribute is bucket for Google Cloud Storage bucket name. When database CRUD operation are performed, corresponding HTTP methods are conditional request are made to the server.   
   
    var schema = {
      stores: [{
        name: 'ydn-db',
        Sync: {
          format: 'gcs', // refer to Google Cloud Storage backend
          immutable: true,
          Options: {
            bucket: 'ydn-test-report-2' // GCS bucket name
          }
        }
      }]
    };
       
### Efficient backend design
       
One thing we concern here is cost or in engineering terms efficiency. Traditional backend server perform basic formatting after querying from the database return only necessary data to the client. REST service are barebone backend service and do not have such luxury. In return, REST service are cheap and easily scalable and portable.

REST API has only two query, GET request on object URI returning the object and GET request on bucket URI, returning list of object URI. Efficient mean we only send request only as necessary to render the view. In terms of cost, GET bucket request does not incur network cost since it does not return any object.

We use client database, YDN-DB, to cache object resource. Whenever a cached data is reused, it must be validated. In REST service, read request cache validation is made by either with [If-None-Match](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.26) etag or [If-Unmodified-Since](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.28) updated date. For write request, [If-Match](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.24) or [If-Modified-Since](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.25) are used to ensure updated record was not modified by others. In this app, we only use read request. We can further reduce the cost in this particular case, since the data is immutable. Once a test is run, the result is written and it cannot be changed. Immutable database does not require cache invalidation. Once we have the data, it can be cached permanently. In YDN-DB, it is done by setting `immutable` attribute to `true`.
       
## REST URI design for querying

In general, we will expect several thousands of results in the bucket. It will be huge cost if we were to cached all of them into client. Worse, most user spend just a brief period. A quick approach will be to display the last, say 25, results. In S3 like REST service, the only query available is ascending order of URI (or primary key). And hence we have to design URI such that last result will come first. This can easily be achieved using negative timestamp from some future epoch.

<table>
    <caption>Typical result table</caption>
    <thead>
    <tr>
        <th>Platform</th>
        <th>Browser</th>
        <th>CRUD</th>
        <th>Cursor</th>
        <th>Event</th>
    </tr>
    </thead>
    <tbody>
    <tr>
        <td>Linux</td>
        <td>Firefox</td>
        <td style="background: lightgreen">Pass</td>
        <td style="background: lightgreen">Pass</td>
        <td style="background: lightgreen">Pass</td>
    </tr>
    <tr>
        <td>Win32</td>
        <td>IE9</td>
        <td style="background: lightgreen">Pass</td>
        <td style="background: red">Fail</td>
        <td style="background: red">Fail</td>
    </tr>
    <tr>
        <td>Win32</td>
        <td>IE9</td>
        <td style="background: lightgreen">Pass</td>
        <td style="background: red">Fail</td>
        <td style="background: red">Fail</td>
    </tr>
    <tr>
        <td>MacIntel</td>
        <td>Safari</td>
        <td style="background: lightgreen">Pass</td>
        <td style="background: red">Fail</td>
        <td style="background: lightgreen">Pass</td>
    </tr>
    </tbody>
</table>

But the last results set does not meet user expectation of feature matrix. User want to see overview. In general there will be repeated platform/browser result, Win32/IE9 in above table. Best information quality is achieved if unique platform/browser are displayed.
       
## Local secondary indexing
       
This problem is typically encountered in using key-value store database. The solution is local secondary indexing as promoted in [Amazon Dynamo API](http://aws.typepad.com/aws/2013/04/local-secondary-indexes-for-amazon-dynamodb.html). To query unique 'platform/browser', we enumerate URI such that resulting URI are ordered by 'platform/browser' value. URI is designed having two parts, the first part is hash predicate and last part is range key, timestamp here. An example URI for our case is:       

    http://http://ydn-test-report-2.storage.googleapis.com/MacIntel/Safari/251988549193682

Unlike database query, enumerating REST URI keys are not straight forward. First, we send GET request to the bucket with [delimiter=/](https://developers.google.com/storage/docs/reference-headers#delimiter), listing all platform available in the bucket. Please note S3 REST API return CommonPrefixes, listing unique 'platform' value here.

    GET http://http://ydn-test-report-2.storage.googleapis.com/?delimiter=/

Then we query again with value of 'platform' as [prefix](https://developers.google.com/storage/docs/reference-headers#prefix), listing all browsers in the platform.

    GET http://http://ydn-test-report-2.storage.googleapis.com/?delimiter=/&prefix=MacIntel

Then we query again with both 'platform/browser' predicate, but limit to one last result using [max-keys](https://developers.google.com/storage/docs/reference-headers#maxkeys).

    GET http://http://ydn-test-report-2.storage.googleapis.com/?prefix=MacIntel/Safari/&max-key=1

Repeating this last query, we enumerate unique 'platform/browser'. This approach is workable, but not an ideal.

### Header metadata indexing
    
Better solution is "header metadata indexing", which is possible in GCS, using [JSON API](https://developers.google.com/storage/docs/json_api/) and [Microsoft Azure Blob Service REST API](http://msdn.microsoft.com/en-us/library/windowsazure/dd135733.aspx). The two ingredients of interest in these newer API are batch query and meta-data in object listing. This is utilized by keeping platform, browser metadata in [custom header](https://developers.google.com/storage/docs/reference-headers#xgoogmeta). The metadata are indexed in the client side database. The app analyze meta data before sending GET request, so that only necessary requests are made.

Additionally, to synchronize, client and server metadata efficiently, we can make URI such that it is always increasing or decreasing. After full data is cached, only newer data can be query using marker key.

This meta data is keep in separate object store as follow:

    var schema = {
      stores: [{
        name: 'ydn-db-meta',
        keyPath: 'name',
        // index meta data in the header
        indexes: [{
            keyPath: 'platform' // x-goog-meta-platform
          }, {
            keyPath: 'browser' // x-goog-meta-browser
          }, {
            keyPath: 'version' // x-goog-meta-version
          }, {
            keyPath: 'etag' // required index for 'ydn-db' store conditional request.
          }, {
            keyPath: 'updated'
          }, {
            // use compound index, so that we can query unique quickly
            name: 'platform, browser',
            keyPath: ['metadata.platform', 'metadata.browser']
          }, {
            // use compound index, so that we can query unique quickly
            name: 'platform, browser, version',
            keyPath: ['metadata.platform', 'metadata.browser', 'metadata.version']
        }],
        Sync: {
          // 'gcs-meta' sync option format store only meta data of the
          // object. The key must be generate in descending order.
          format: 'gcs-meta',
          // prefetch only 'meta', other possible is 'full'
          // 'meta' is default for 'gcs-meta' sync format.
          prefetch: 'meta',
          // Prefetch refractory period interval in milliseconds.
          prefetchRefractoryPeriod: 60 * 1000, // Default is 5000.
          Options: {
            bucket: 'ydn-test-report-2',
            prefix: 'ydn-db/'  // path prefix for this store.
          }
        }
      }]
    };
    
## An angular controller for database query
   
The job of a controller is preparing data to display in view. This involves querying from the database and formatting the result into model data suitable for rendering them in views.

The home page of this app is feature matrix of unit test result from unique set of browsers. To query unique browser, a compound index `platform`, `browser` is created on `ydn-db-meta` store. Using the index, we can query or enumerate unqiue unique secondary key. Since we keep primary key the same for both `ydn-db-meta` and `ydn-db`, it is used to retrieve result set from `ydn-db` object store.

    angular.module('myApp.controllers', [])
        .controller('HomeCtrl', ['$scope', 'utils', 'database',
            function($scope, utils, db) {
              var index_name = 'platform, browser';
              var key_range = null;
              var limit = 50;
              var offset = 0;
              var reverse = false;
              var unique = true;
              db.keys('ydn-db-meta', index_name, key_range, limit, offset, reverse, unique)
                .then(function(keys) { // list of primary key for unique browser
                  var req = db.values('ydn-db', keys);
                  req.then(function(json) {
                  $scope.results = utils.processResult(json);
                  $scope.$apply(); // pump angular event queue
                }, function(e) {
                  throw e;
            }, this);
        });
    }])
    
Resulting data are further processed in `utils.processResult` service function. This utility function transform raw qunit output into array of objects, `results`, suitable for rendering into a table in the controller template with the help of `resultView` directive.
    
    
    <table class="feature">
        <thead>
        <tr>
            <td width="20%">Platform</td>
            <td width="20%">Browser</td>
            <td width="10%">CRUD</td>
            <td width="10%">Cursor</td>
            <td width="10%">Event</td>
            <td width="10%">Transaction</td>
            <td width="10%">Query</td>
            <td width="10%">SQL</td>
        </tr>
        </thead>
        <tbody>
        <tr ng-repeat="resultSet in results">
            <td>{{resultSet.platform}}</td>
            <td>{{resultSet.browser}} - {{resultSet.version}}</td>
            <td><span class="cell"><a result-view name="crud" ></a></span></td>
            <td><span class="cell"><a result-view name="cursor"  ></a></span></td>
            <td><span class="cell"><a result-view name="event"  ></a></span></td>
            <td><span class="cell"><a result-view name="transaction"  ></a></span></td>
            <td><span class="cell"><a result-view name="query"  ></a></span></td>
            <td><span class="cell"><a result-view name="sql"  ></a></span></td>
        </tr>
        </tbody>
    </table>
        
## Security Model
   
This application do not require authentication. Unit test results are supposed to collect anonymously so that privacy are observed. Anyone can POST (create new data) or PUT (override) existing data. It is possible to prevent overriding by changing ACL during create the object. Abuse of data is mainly relied on browser cross origin policy. Even though bucket access anonymous write access, since CORS is granted to selected origins, other web site cannot write to the bucket however.

## Browser requirement

For security reason, data and web site cannot be in same origin. The app is running on trusted origin and all HTML contents (and hence js and css) are secured. Data read from the bucket are taken as untrusted resources. Data from untrusted sources are not directly executed, but parsed to JSON object.

To send the data we need to write POST cross origin to REST server. Sending POST request to different host is only allowed by [form post](http://www.w3.org/TR/html401/interact/forms.html#h-17.13.4), which has content type of `application/x-www-form-urlencoded`. It is different from what we want of JSON format. Additionally form post do not send custom headers, which is require for our application. The solution is to use newer XMLHttpRequest level 2 as part of [Cross-Origin Resource Sharing](http://www.w3.org/TR/cors/). Most modern browser support CORS, but still have problem in Safari regarding to content type. To make problem worse, our application need to support browser including IE6 for our unit test. There is no way, but to use proxy server. I have use Google appengine for proxying form data. Google appengine is not only generous free tier, GCS integration is very easy.        
    

## Questions?

Please check out running [demo app](http://dev.yathit.com/demo/feature-matrix/index.html) to understand how the app work. Full source code is available in [Github](https://github.com/yathit/feature-matrix). To replicate the repo, you will need to create Google Cloud Storage bucket and configure as described in the project readme file.

If you are not clear about how or why, feel free to make a comment below. For bug report of the source code, please fine issue on github. For alternative or better idea, please discuss and send pull request.

[Feature matrix Angular.js app](/demo/feature-matrix/index.html) ([source code repo](https://github.com/yathit/feature-matrix))

{% endwrap %}     