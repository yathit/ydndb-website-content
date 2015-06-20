---
layout: ydndb-article
description: "NoSQL query"
class: ydndb
title: "Compound index"
introduction: "The IndexedDB API provides compound indexes to make complex queries possible."
article:
  written_on: 2013-04-01
  updated_on: 2014-04-28
  order: 2
collection: ydndb-query
authors:
  - kyawtun
notes:
  messages:
    - Pages in this section include the YDN-DB script and some preloaded data and utility functions.  You can follow the sample code in your browser's developer console to see in action.
    - Your database will have to be loaded with sample data as described in section home page.
    
---

{% wrap content %}

A [compound key](http://en.wikipedia.org/wiki/Compound_key) is a key that consists of two or more attributes that uniquely identify an entity occurrence. In IndexedDB, a compound key can be created in an index from either a [keyPath](http://www.w3.org/TR/IndexedDB/#key-path-construct) array or from a  [key](http://www.w3.org/TR/IndexedDB/#key-construct) array as described in [Nested Set Model](../setup/key.html#nested-set-model). 

A compound index is created by providing a keyPath array in the [schema store](/api/ydn/db/schema.html#Store) section. For example, the [data-seeding.js](/js/ydn-db/data-seeding.js) file's `'company_store_schema'` uses compound indexes extensively for querying. 

{% include modules/remember.liquid title="Tip" inline=true list=page.notes.messages %}

Suppose you want to filter the 'license' and 'publisher' fields of the 'article' store.  You can index both fields into a composite index by specifying a keyPath array as follows:

    var schema = {
      store: [{
        name: 'article',
        indexes: [{
          name: 'license, publisher',
          keyPath: ['license', 'publisher']
        }]
      }]
    };

The index value is an array of two elements: the first element is the 'license' field and the second element is the 'publisher' field of the record. The key comparison algorithm ensures that results are sorted by license followed by publisher and iterates through the index in this order. If we filter 'license' to 'SA', we get the results sorted by publisher as follows:
    
    key_range = ydn.db.KeyRange.starts(['SA']);
    db.values(new ydn.db.IndexValueIterator('article', 'license, publisher', key_range), 10).done(function(articles) {
        console.log(articles); 
      }
    );
        
Notice that we keep the 'license' field constant to get the 'publisher' field sorted.   Alternately, we can filter 'publisher' by range.      
  
    key_range = ydn.db.KeyRange.bound(['SA', 'A'], ['SA', 'B']);
    db.values(new ydn.db.IndexValueIterator('article', 'license, publisher', key_range)).done(function(articles) {
        console.log(articles); 
      }
    );
    
Only the last field in an index can be filtered by a range in index query. Keep this limitation in mind.

It is not possible to use this index to filter by 'publisher' and sort by 'license'. That requires another index of `['publisher', 'license']`. This limits compound index use in ad hoc queries.
 
Note that the list method does not have an offset when using an iterator. This is by design to improve performance and robustness: skipping a number of records is not fast for a large offset value. Also, you cannot be sure that the number of records does not change during execution of the query. The iterator saves the current position so it can resume from that position.  If you are using views to page through results, invoke the list again with the existing iterator to page to the next view. The position can even be be persisted to storage and resumed at a later time. To reverse the direction of the results, use `reverse` method of the iterator.
 
Queries using a compound index take only logarithmic time on object store records: it is very fast.  Writing data with indexes requires more CPU time and extra storage for the indexes. Queries have to be planned before indexing and queries are limited to one range filter.  Multiple sort orders are supported.
     
{% endwrap %}       
