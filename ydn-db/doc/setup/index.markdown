---
layout: ydndb-section
description: "Getting started with YDN-DB javascript library and fundamental database concepts"
class: ydndb
title: Getting started
introduction: "This section illustrate basic usage of the library and explains fundamental database concepts."
article:
  written_on: 2013-04-01
  updated_on: 2014-04-28
  order: 1
collection: ydndb-doc
id: ydndb-setup
notes:
  console:
    Pages in this sections include the YDN-DB script and some preloaded data and utility functions, so that you follow the sample code in your browser's developer console to see in action.
  schema:
    Always specify database schema on any non-trivial web application. Database schema should not be changed during a life time of an running applicaton.
  auto:
    Auto-versioning is preferred way of YDN-DB storage instance.
g_comments_href:
  http://dev.yathit.com/ydn-db/getting-started.html
---

{% wrap content %}

## How to use the library

YDN-DB is a pure javascript and hence simple include a pre-build file, such as [ydn.db-dev.js](/jsc/ydn.db-dev.js) in this page, to your HTML document before using the library. Pre-build files are available in [download section](/ydn-db/downloads.html). Any one of the file will do and only one file is required. Each file are built for specified feature set. For more details about these files, see in [setup article](setup.html).

{% include modules/remember.liquid title="Tip" inline=true text=page.notes.console %}


## Database connection

A simple way to initialize a database is by specifying a database
  name.

    db = new ydn.db.Storage('db-name');
    db.put('store-name', {message: 'Hello world!'}, 'id1');
    db.get('store-name', 'id1').always(function(record) {
      console.log(record);
    });

The storage instance, `ydn.db.Storage`, connects to suitable data storage mechanisms ranging from IndexedDB to WebSQL to localStore depending on browser. It will open existing database or create a new database
  with the given database name.

All database operation methods are asynchronous and result are available as a [*thenble promise* object](https://github.com/promises-aplus/promises-spec). The resulting promise object accepts two asynchronous functions: `done` to receive value on successful request and `fail` to received in case of error. More commonly `then` method can be used to receive result of both of cases. In case of error, this library always invokes with `fail` callback with `Error` object with has `name` and `message` for Error name and its description respectively. Generally stack trace, `error.stack` are available.

## Storing data

Use `put` method to insert a new or update existing
  record(s).

    db.put('store1', {test: 'Hello World!'}, 123).then(function(key) {
      console.log(key);
    }, function(e) {
      console.error(e.stack);
    });

The first argument is store name. It is [object store](http://www.w3.org/TR/IndexedDB/#object-store-concept) name in IndexedDB and TABLE name in WebSQL. Since a schema is not given, a table or object store will be created if not exist.

The second argument is record value that we want to store in the database. It should be a simple javascript object ((JSON)[http://json.org/]). A [*structured clone*](http://www.w3.org/TR/html5/common-dom-interfaces.html#internal-structured-cloning-algorithm) of the object is stored in the database. A structured clone is similar concept in JSON serialization, but it is more efficient and more powerful. File and Blob are serializable. If the record fail to clone it, underlying database API shall throw uncaught `DataCloneError`.

The third argument is *primary key* of the record. A [key](http://www.w3.org/TR/IndexedDB/#key-construct) can be number, string, Date or array of those types. Since we are given primary key separately from the record, it is called *out-of-line* key.

Use `get` method to retrieve it by the primary key.

    req = db.get('store1', 123);
    req.done(function(record) {
      console.log(record);
    });
    req.fail(function(e) {
      console.log(e.message);
    });


## Retrieving

Keys are the *most* efficient way to retrieve a record. If we don't know the key, we *must* enumerate the
  whole store to find it.

Let us add some more records to the store.

    var data = [{message: 'a record'}, {message: 'b record'}];
    db.put('store1', data, ['a', 'b']).always(function(x) {
      console.log(x);
    });

Notice multiple records are stored by using array of records in single [*database transaction*](http://en.wikipedia.org/wiki/Database_transaction).

Record values are retrieved by using `values` *database operation method*.

    db.values('store1').always(function(records) {
      console.log(records);
    });

We can also retrieve only primary key of the records using `keys` database operation method.

    db.keys('store1').done(function(records) {
      console.log(records);
    });

In contrast to retrieving record values, key only retrieval is much faster because it obviates record value de-serialization. Key is very important for effective querying, and hence keys should be carefully constructed. In addition to primary key, there is *secondary key*, which is simply called as *index key*.


## Database Schema

While running the above codes, we modified database schema on creating new stores. It is not preferable in production usage, because modifying database schema is not a trivial process. It need to notified all connections on other tabs including worker thread as well. Additionally we should use a fixed schema through out a web application for consistency.

{% include modules/remember.liquid title="Note" inline=true text=page.notes.schema %}

The database schema from the database connection is retrieved as follow.

    db.getSchema(function(schema) {
      console.log(schema);
    });

You will find database version is `undefined`, since we are not giving a database version. The database is said to be in *auto-version* mode.

{% include modules/remember.liquid title="Note" inline=true text=page.notes.auto %}


A [`database schema`](/api/ydn/db/schema.html) is define object stores or TABLE(s) in WebSQL. As an example:

    var author_store_schema = {
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
        }
      ] // optional, list of index schema as array.
    };
    schema = {
      stores: [author_store_schema]
    };

The above schema define one object store. The `name` of the object store is 'author'. Since <code>keyPath</code> is defined, it is using in-line key. Since `autoIncrement` is `false`, all records must have a valid key in its 'email' field attribute. See more detail about [database schema article](schema.html).

The object store 'author' has three <em>secondary indexes</em>,
  namely 'born', 'company' and 'hobby'. In WebSQL, they are column names
  of TABLE 'author'.
  If <code>keyPath</code> is not defined, it is default to <code>name</code>.
  An example 'author' record will be:

    author_1 = {
      email: 'me@aaronsw.com',
      born: 531763200000,
      first: 'Aaron',
      last: 'Swartz',
      company: 'Reddit',
      hobby: ['programming', 'blogging', 'politics']
    };

If index schema attribute, `unique` is `true`, unique constraint is applied on the index key. If unique constraint is void during a database write operation, [`ConstriantError`](http://www.w3.org/TR/IndexedDB/#dfn-constrainterror)
  will be issued.

The index schema attribute, `multiEntry` is meaningful only for key value of array data type. The index 'hobby' has `multiEntry` of `true`, so that each element in of the array hobby are indexed individually.

In addition to 'stores' attribute, database schema take 'version'
  attribute. If version number is specified, the library will open with
  the given version. If the client browser do not have or lower than the
  given version, it will be upgraded as necessary. Client version must
  not be higher than given version. If client version is the same as
  given version, the database schema must be similar. If not similar,
  the library will refuse to connect the database. This library will not
  work, if schema is not known.

Let us generate some data for querying.

    genAuthors = function(n) {
      var out = [];
      for (var i = 0; i < n; i++) {
        out[i] = {
          first: ydn.testing.randName(),
          last: ydn.testing.randName(),
          born: +(new Date(1900+Math.random()*70, 12*Math.random(), 30*Math.random())),
          email: ydn.testing.randEmail(),
          company: pickOne(companyList),
          hobby: pickMany(hobbyList)
        };
      }
      return out;
    };

    db = new ydn.db.Storage('test-2', schema);
    var authors = genAuthors(10000);
    db.put('author', authors).then(
      function(ids) {
        console.log(ids.length + ' authors put.');
      }, function(e) {
        console.log(e.message || e);
      }
    );

## Basic query

### Counting

Use `count` method to count number of record in a store or a continuous portion o store.

    db.count('author').done(function(x) {
      console.log('Number of authors: ' + x);
    });

This is the only aggregate database method provided by the IndexedDB API.

### Sorting

Keys are sorted in the database and hence database query results are *always* sorted in some way. By default, the following query is sorted by primary key, 'email'.

    var key_range = null;
    db.from('author').list(10).done(function(records) {
      console.log(records);
    });

If sorting is required, the sorted field have to be indexed. The following illustrate iterating records sorted by 'born' date field, one of the three indexed fields.

    db.from('author').order('born').list(10).done(function(records) {
      console.log(records);
    });
    db.from('author').order('born').reverse().list(10).done(function(records) {
      console.log(records);
    });

The following example retrieve list of unique hobbies.

    db.from('author').select('hobby').unique(true).list().done(function(hobby) {
      console.log(hobby);
    });

### Filtering

The primary way of filtering is query by [key range](/api/ydn/db/keyrange.html). More sophisticated filtering are iterated merging of key range results. We dedicate these sophisticated filtering on [query section](../query/index.html).

The following query finds authors born in 1942 February.

    var lower = + new Date(1942, 1, 1); // 1942 February 1
    var upper = + new Date(1942, 2, 1); // 1942 March 1
    db.from('author').where('born', '>=', lower, '<', upper).list().done(function(records) {
      console.log(records);
      records.map(function(x) {
        console.log(x.first + ' ' + x.last + ' ' + new Date(x.born));
      });
    });

## Updating

Use `open` method to update records.

    var iter = new ydn.db.ValueIterator('author', ydn.db.KeyRange.starts('a'));
    var mode = 'readwrite';
    var updated = 0;
    var deleted = 0;
    db.open(function(cursor) {
      var author = cursor.getValue();
      if (author.company == 'Oracle') {
        cursor.clear().done(function(e) {
          deleted++;
        });
      } else if (author.category != 'A') {
        author.category = 'A';
        cursor.update(author).done(function(e) {
          updated++;
        });
      }
    }, iter, mode).then(function() {
      console.log(updated + ' records updated, ' + deleted + ' deleted.');
    });

{% endwrap %}

<script src="/js/ydn-db/data-seeding.js"></script>
