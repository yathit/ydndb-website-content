---
layout: ydndb-article
description: "Database Schema"
class: ydndb
title: Schema
introduction: "Database schema define how your data are stored and retrieved."
article:
  written_on: 2013-04-01
  updated_on: 2014-04-28
  order: 3
collection: ydndb-setup
authors:
  - kyawtun
notes:
  schema:
    Always specify database schema on any non-trivial web application. Database schema should not be changed during a life time of an running application. 
  auto:
    Auto-versioning is preferred way of YDN-DB storage instance.  
---

{% wrap content %}

## IndexedDB database schema

An [IndexedDB](http://www.w3.org/TR/IndexedDB/) database comprises collection of [object stores](http://www.w3.org/TR/IndexedDB/#object-store-concept), which in turn, comprises collection of [indexes](http://www.w3.org/TR/IndexedDB/#object-store-concept). In relational database terms, these two represent TABLE and COLUMN. In YDN-DB, database schema definition of object store along with its indexes, [full-text catalog option](../full-text/index.html) and [synchronization option](../sync/index.html). The last two options change schema by adding or modifying object store(s) definition to meet their requirement.
     
During [opening](http://www.w3.org/TR/IndexedDB/#widl-IDBFactory-open-IDBOpenDBRequest-DOMString-name-unsigned-long-long-version) of [database connection](http://www.w3.org/TR/IndexedDB/#dfn-connection), object stores and indexes must be created in [upgradeneeded phase](http://www.w3.org/TR/IndexedDB/#dfn-steps-for-running-a-versionchange-transaction), if they are not already exist. Existing object store and their indexes can be query synchronously from the database connection. Upgradedneed phase, `onupgradeneeded` callback on [IDBOpenDBRequest](http://www.w3.org/TR/IndexedDB/#idl-def-IDBOpenDBRequest) is invoked only when a connection is opened with higher than existing version.

## YDN-DB database schema

YDN-DB library create object stores along with their indexes during database connection as defined in the YDN-DB database schema. [YDN-DB database schema](/api/ydn/db/schema.html#database) is defined by an JSON object. For example:

    var schema = {
      version: 1,
      autoSchema: false, // must be false when version is defined
      stores: [{
        name: 'author',
        keyPath: 'email', // optional,
        autoIncrement: false, // optional.
        indexes: [
          {
            name: 'born', // optional
            keyPath: 'born',
            unique: false, // optional, default to false
            multiEntry: false // optional, default to false
          }, {
            name: 'company'
          }, {
            name: 'hobby',
            multiEntry: true
          }, {
            name: 'first-name',
            keyPath: 'name.first'
          }, {
            name: 'full-name',
            keyPath: ['name.first', 'name.last']
          }
        ], // optional, list of index schema as array.
        Sync: {
          format: 'gcs',
          Options: {
            bucket: 'ydn-data1',
            prefix: 'author/'
          }
        }
      ],
      fullTextCatalogs: {
        name: 'author-name',
        sources: [{
          storeName: 'author',
          keyPath: 'first',
          weight: 1.0
        }, {
          storeName: 'author',
          keyPath: 'last',
          weight: 0.8
      }]
    };
    
### Database version   
    
`version` attribute in YDN-DB database schema refer to [IndexedDB version](http://www.w3.org/TR/IndexedDB/#dfn-version). If defined in the schema, it is used on database `open` method. `version` value must be positive integer. Generally `version` value is increased by 1 if an schema change in new version of the application. If `version` attribute in YDN-DB database schema is not defined, it is said to be *auto version* mode. In auto version mode, the library increase database version by 1 if existing database schema is different from YDN-DB database schema defined.   
   
{% include modules/remember.liquid title="Tip" inline=true text=page.notes.auto %}
   
Database version must not be lower than existing database version. A way to know existing database version is opening the database without giving version. Generally we do not need to know existing database version before opening database, because as application author, we know all versions we used before. Existing database version in the user browser must be one of these versions used before. But in practice, if an application depend on specific database version, managing database version can be quite complicate over years of refactoring the application. Whenever possible, use auto version.
    
An application which require to do some logic during database changes can listen [`ready` event](/api/ydn/db/events.html#ready). The open event listener is invoked with custom event which has `getVersion()` and `getOldVersion()` methods to query new and old database versions.    

### YDN-DB store schema

YDN-DB database schema defines a collection of object stores in `stores` attribute. Each element of `stores` define a store representing an [IndexedDB object store](http://www.w3.org/TR/IndexedDB/#object-store). It is similar to TABLE in relational database. The main different is, object stores are unrelated to other object stores where as TABLE can be related to other TABLE by [foreign key](http://en.wikipedia.org/wiki/Foreign_key) constraint. Object store is identified by its [`name`](http://www.w3.org/TR/IndexedDB/#dfn-object-store-name) attribute.

*Object store* store records, which comprises a _value_ and a _key_, primary key. If key is not provided during storing record, a key will be generated by the database, if `autoIncrement` attribute is set to `true`. In this case, the object store is said to be using [key generator](http://www.w3.org/TR/IndexedDB/#dfn-key-generator). This is similar to [`AUTO_INCREMENT`](http://dev.mysql.com/doc/refman/5.0/en/example-auto-increment.html) in relation database. 
    
Key can be specified _either_ in the record value or externally, refers to [*in-line*](http://www.w3.org/TR/IndexedDB/#dfn-in-line-keys)  or [*out-of-line*](http://www.w3.org/TR/IndexedDB/#dfn-out-of-line-keys) key respectively. An object store using *in-line* key specify [`keyPath`](http://www.w3.org/TR/IndexedDB/#dfn-key-path) attribute for extracting key from the record value. Dotted path can be used to extract key nested deeper in the object. For example, the keyPath of `name.first` will extract 'Foo' of the following object.
     
     {
       name: {
         first: 'Foo',
         last: 'Bar'
       },
       age: 20
     }

Out-of-line key are specified during storing record as follow:

    db.put('store1', file, 123);
    
It is preferable to use in-line key over out-of-line key for simplicity.

The above schema define one object store. The `name` of the object store is 'author'. Since `keyPath` is defined, it is using in-line key. Since `autoIncrement` is `false`, all records must have a valid key in its 'email' field attribute. 

An object store has a collection of indexes to specify secondary index, as described in next section.

In additional to these intrinsic properties in object store schema, `Sync` and `fullTextCatalogs` can be defined to modified database schema. They are explained in respective [synchronization](../sync/) and [full-text search](../full-text) sections.

### Auto schema

The library also allow to change database schema during life time of an application, by setting `autoSchema` to `true`. New object store or index are created as necessary. 

An out-of-line key object store, named as 'store1' will be created by:

    db.put('store1', {msg: 'test'}, 123);
    
An in-line key object store, named as 'store2' will be created with given key path as follow:
    
    db.put({name: 'store2', keyPath: 'id'}, {id: 456, msg: 'test'});
    
If require object store is not existed, the database connection is closed and a new database connection is created with updated schema. These database update processes are costly and unreliable.    

{% include modules/remember.liquid title="Caution" inline=true text=page.notes.schema %}

### YDN-DB index schema

If you need to query other than primary key, index must be created by defining in index schema.
    
Above object store 'author' has five *indexes*, namely 'born', 'company', 'first-name', 'full-name' and 'hobby'. In WebSQL, they are column names of TABLE 'author'. If `keyPath` is not defined, it is default to `name`.
  
An example 'author' record will be: 
  
    author_1 = {
      email: 'me@aaronsw.com',
      born: 531763200000,
      name: {
        first: 'Aaron',
        last: 'Swartz',
      }
      company: 'Reddit',
      hobby: ['programming', 'blogging', 'politics']
    };
    
If index schema attribute, `unique` is `true`, unique constraint is applied on the index key. If unique constraint is void during a database write operation, [`ConstraintError`](http://www.w3.org/TR/IndexedDB/#dfn-constrainterror)
  will be issued in reject callback of database request promise.
   
When `keyPath` is defined as an array, they key compose of more than one value forming a [compound key](http://en.wikipedia.org/wiki/Compound_key). Designing compound key plays critical role for [key joining operation](../query/key-join.html) and other interesting [query](../query/compound-key.html). For record value `author_1`, the index value of 'full-name' is evaluated as "['Aaron', 'Swartz']".    
  
The index schema attribute, `multiEntry` is meaningful only for key value of array data type. The index 'hobby' has `multiEntry` of `true`, so that each element in of the array hobby are indexed individually.


{% endwrap %} 