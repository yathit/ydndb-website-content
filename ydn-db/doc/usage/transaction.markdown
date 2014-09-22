---
layout: ydndb-article
title: "Transaction"
introduction: "Although not apparent, transaction is the most elaborated part of YDN-DB"
article:
  written_on: 2014-09-5
  updated_on: 2014-09-5
  order: 2
collection: ydndb-using
authors:
  - kyawtun
g_comments_href:
  http://dev.yathit.com/ydn-db/transaction.html
---

{% wrap content %}

Currently client-side database are [temporary storage](http://www.w3.org/TR/quota-api/#temporary). The data can be wipe out anytime without informing to user or script. Data corruption instance has been observed to SQLite and Chrome IndexedDB during opening the database. If corruption occur, whole database or only certain object store can wipe out.   

The most unpredictable operation is database connection. For an application to robust, [storage events](http://dev.yathit.com/api-reference/ydn-db/storage.html) dispatched from the storage instance should be listened and validate data integrity.
    
    var db_name = 'test-db-3';
    var options = {};
    db = new ydn.db.Storage(db_name, author_article_topic_schema, options);
    db.addEventListener('ready', function(ev) {
      if (isNaN(ev.getOldVersion())) {
        // new database is created
        var authors = genAuthors(100);
        db.put('author', authors);
      } else if (ev.getVersion() > ev.getOldVersion()) {
        // schema upgrade, do data upgrade as necessary
      } else {
        // existing database open
      }
    });
    db.addEventListener('fail', function(ev) {
      var err = ev.getError();
      // database connection fail, inform user
      alert('Your data will not be saved, database opening failed wth ' + err.name);
    });
        
## Transaction policy
  
###  Parallel and serial transaction thread
  
This library hides the transaction layer for ease of use. Each database operation method requires an active transaction. The transaction could be created or reused. The selection is determined by transaction policy. Multiple transactions can be created from a connection. Non blocking transactions are execute in parallel. For example all readonly transactions are executed in parallel. Executing parallel has performance benefit, however we may want to preserve order. This library provide transaction thread of either parallel or serial. 
  
Consider for populating a store with records from a large delimited text. To avoid holding memory for holding all records, data are download by chunk using HTTP Range header by `CsvStreamer` and invoke next callback when it read a record. A chunk of data comprise multiple records and in that case `CsvStreamer` write synchronously. While waiting for next chunk of data CsvStreamer invoke next callback asynchronously.  
        
    var stream = new CsvStreamer(url);
    var isSerial = false;
    var tdb = db.branch('multi', isSerial); // multi-request parallel transactions
    var putData = function(data) {
      if (data) { 
        tdb.put('store1', data).then(function() {
          stream.next(function (data) {   
            putData(data);
        }), function(e) {
          console.log(e.message || e);
        });     
      }
    });
     
    stream.next(function (data) {
      putData(data);
    }
            
In this example a new multi-request parallel transaction thread is created using branch method. This kind of transaction thread suitable for this use case, in which synchronous write use existing transaction and asynchronous write use parallel transaction. 

A serial transaction thread is created by using [branch](/api/ydn/db/storage.html#branch) method with `isSerial` flag to `true`. In serial transaction thread, a new transaction is created only after active transaction is committed. Transaction request are queue and popped by last-in-fast-out basic. The default transaction thread, which is attached to the storage instance, is serial. Serial transaction thread is used when order of execution is important.

### Request

At the same time, multiple requests can place into a transaction. These request are executed in order in the transaction it placed. Creating a new transaction has higher cost than creating a new request. Hence placing multiple requests on a transaction has huge performance over running multiple transactions. 

*Single request policy* (`'single'`): Each database request methods create a new request. If request is active, the creating process is pushed into a queue. Whenever a request is completed the queue is popped by last-in-first-out basic. This method satisfy read-your-own-write consistency.

*Atomic request policy* (`'atomic'`): Atomic serial queue transaction policy create transaction as in serial queue transaction thread policy and intersect database request method by holding result until transaction is committed.

If the request failed, when transaction is aborted. The database request method cannot [abort](/api/ydn/db/request.html#abort) the transaction. 

*Multi-request policy* (`'multi'`): A new transaction is created if no active transaction or the active transaction scope is not applicable for the requested database operation method. If transaction is active but not reused, the creating process is pushed into a queue. Whenever a transaction is completed the queue is popped by last-in-first-out basic.

*Repeat request policy* (`'repeat'`): This is similar to multi-request policy but restrict to same transaction scope.

*Explicit request policy* (`''`): Transaction is created by using explicit scope.

### Explicit transaction

Transaction is used to achieve atomic, consistent, isolated database operation. In this library, all database operation methods are performed in an implicit transaction. However, sometimes, explicit transaction are desirable. Although you can create an explicit transaction, its commit still be implicit. None of the native API in IndexedDB nor WebSQL provides to commit a transaction explicitly.

#### For consistency

An explicit transaction is performed by obtaining transaction database instance using run method. This example demonstrates one use of transactions: updating an entity with a new property value relative to its current value.         
    amount = 10;
    db.run(function health_10up(tdb) { // tdb is transaction database instance
       tdb.get('player', 1).done(function(p1_obj) {
            p1_obj.health += amount;
            tdb.put('player', p1_obj);
       });  
    }, ['player'], 'readwrite');
       
This requires a transaction because the value may be updated by another user after this code fetches the object, but before it saves the modified object. Without a transaction, the user's request uses the value of health prior to the other user's update, and the save overwrites the new value. With a transaction, the application is told about the other user's update. If the entity is updated during the transaction, then the transaction is retried until all steps are completed without interruption.

If operation fail, the transaction is abort and relevant `error` callback of the key will be invoked. As best practice, keep transaction operation short and quick and resolve error in early possible step.

This arrangement provides very natural and robust (meaning that _never_ you will encounter `InvalidStateError`) transaction workflow on most web application requirement.      

#### Ensuring write

The success callback in all database operation methods are invoked by request success event. For a write database operation, a request success does not grantee that data is written because its transaction it not committed yet. The transaction commit can fail. To ensure database write, the transaction completed can be listen using explicit transaction.  
     
    db.run(function health_10up(tdb) { 
       var req = tdb.put('player', obj).done(function(key) {
        // put request is success, but not grantee written to the database
        req.abort(); // we can still abort the request
       });  
    }, ['player'], 'readwrite'function on_completed(type, e) {
      if (type == 'complete') {
        console.log('data written to the database.');
      } else if (type == 'abort') {
        console.log('aborted');
      } else { // must be an error
        console.log(e.message || e);
      }
    });     
    
### For performance

Creating a transaction is relatively expensive in compare to making a request. Depending on transaction policy, each request may not reuse the available active transaction. The following code explicitly create a transaction re-used in the loop.    

Prellel transaction thread create a new transaction immediate, where as in serial transaction thread, transaction are created only if previous transaction is committed. Overflow policy request to reuse the previous transaction if compatible.

It should be noted that the above code avoid using large synchronous loop. Instead asynchronous streaming is used.

   
{% endwrap %} 