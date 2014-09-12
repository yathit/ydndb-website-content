---
layout: ydndb-article
description: "Database Key"
title: "Events"
introduction: "YDN-DB event module dispatch useful database events."
article:
  written_on: 2013-04-01
  updated_on: 2014-04-28
  order: 5
collection: ydndb-setup
authors:
  - kyawtun
notes:
  console:
    Pages in this sections include the YDN-DB script and some preloaded data and utility functions, so that you follow the sample code in your browser's developer console to see in action.

---

{% wrap content %}

### YDN-DB event module

YDN-DB is composed of [several modules](setup.html#ydn-db-modules) and one of them is Event module. With event module, such as 'ydn.db-iswu-core-e-qry-dev.js', database instance dispatches four types of [events](/api/ydn/db/events.html). Responding to these events are necessary to handle very rare problem during database schema upgrading. 
 
### Database ready event
 
When storage instance is connected and ready for use, it dispatch `'ready'` [StorageEvent](/api/ydn/db/events.html#StoreEvent). `'ready'` event is dispatch only once before connection time out interval (default to 30 seconds). All database requests must wait until `'ready'` state and push in request queue. Hence heavy database operation are requested only after ready as follow: 


    db.addEventListener('ready', function (event) {
      var is_updated = event.getVersion() != event.getOldVersion();
      if (is_updated) {
        console.log('database connected with new schema');
      } else if (isNaN(event.getOldVersion()))  {
        console.log('new database created');
      } else {
        console.log('existing database connected');
      }
      // heavy database operations should start from this.
    );

{% include modules/remember.liquid title="Tip" inline=true text="Instead of listening 'ready' event, `onReady()` can be used as well." %}

The `'ready'` `StorageEvent`, has `getVersion()` and `getOldVersion()` methods to get current and old database version. If a new database is created, old version is `NaN`. If the existing database is upgraded to new schema, these two versions are different. By checking these values, migration logic can be performed.

### Database fail event

Storage instance dispatches `'fail'` [StorageErrorEvent](/api/ydn/db/events.html#StorageErrorEvent) when it no longer be used due to fatal error. The event is the last event dispatched. Database request cannot be placed during and after this event. 

    db.addEventListener('fail', function (event) {
      var err = event.getError();
      console.log('connection failed with ' + err.name + ' by ' + err.message);
      db = null; // no operation can be placed to the database instance
    });
    


### Database version change event 
    
When an app, possibly from other tab, open with new version of the database current event receive [IDBVersionChangeEvent](http://www.w3.org/TR/IndexedDB/#idl-def-IDBVersionChangeEvent). The default operation of the event is closing the connection.    

An app may want to store data before the connection was closed. This is the last chance to to use the connection. Database request can still be execute at this time, for example to save draft data. If database request have to be make asynchronously, such as sending request to server, default close operation can be prevented by invoking `event.preventDefault()`. After executing the request, database connection must be closed `db.close()` so that other pending connection can be established. 

    db.addEventListener('versionchange', function(e) {
      db.put('temp', draft_data);
    });
     
### Database error event     

Storage instance dispatches `'error'` `StorageErrorEvent` received from the storage mechanism. This event is similar to `window.onerror`, and suppose to be bubble up to global `window`. In IndexedDB API v2, the event will be bubble up to `window` object. Application logic should not be placed under this listener. Event handler should be used only for debugging and accounting purpose.

    db.addEventListener('error', function (event) {
      var e = event.getError(); 
      // common errors are AbortError, ConstraintError and UnknownError (possibliy for Quota exceed error).
      // log error for debugging
      console.log('connection failed with ' + e.name);
    });
    
### Installable events
    
When a record is created, modified or deleted from an object store, events can be requested to be emitted from the storage instance. These events, [StoreEvent](/api/ydn/db/events.html#StoreEvent] and [RecordEvent](/api/ydn/db/events.html#RecordEvent], are installed by setting `dispatchEvents` to `true` on [store schema](/api/ydn/db/schema.html#Store).
   
   
    var schema = {
      stores: [{
        name: 'store1',
        dispatchEvents: true
      }]
    }
    var db = new ydn.db.Storage('event store', schema);
    db.addEventListener(['created', 'updated', 'deleted'], function(event) {
      var keys = [];
      if (event instanceof ydn.db.events.StoreEvent) {
        keys = event.getKeys();
      } else {
        // ydn.db.events.RecordEvent
        keys.push(event.getKey);
      }
      // process keys
    });
    
But note that due to requirement for extra query, current implementation cannot distinguish between record created and record updated event.    
    
In busy database, these event are noisy and tend to be expensive. Probably in future IndexedDB specification may support build-in event.
    
{% include modules/remember.liquid title="Caution" inline=true text="Future version of the library may not support installable events. Application can, and should, be designed without using these events." %}
    

{% endwrap %} 