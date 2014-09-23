---
layout: ydndb-article
description: "NoSQL query"
class: ydndb
title: "Compound index"
introduction: "IndexedDB API provides compound index to make complex query possible."
article:
  written_on: 2013-04-01
  updated_on: 2014-04-28
  order: 2
collection: ydndb-query
authors:
  - kyawtun
notes:
  messages:
    - Pages in this sections include the YDN-DB script and some preloaded data and utility functions, so that you follow the sample code in your browser's developer console to see in action.
    - Database have to be loaded with sample data as described in section home page.
    
---

{% wrap content %}

A [compound key](http://en.wikipedia.org/wiki/Compound_key) is a key that consists of two or more attributes that uniquely identify an entity occurrence. In IndexedDB, compound key can be created either by compound index using array [keyPath](http://www.w3.org/TR/IndexedDB/#key-path-construct) or [key](http://www.w3.org/TR/IndexedDB/#key-construct) itself is an array as described in [nested set model](../setup/key.html#nested-set-model). 

Compound index is created by providing keyPath as array in [store schema](/api/ydn/db/schema.html#Store). See [data-seeding.js](/js/ydn-db/data-seeding.js) file, in which `'company_store_schema'` extensively use compound index for querying. 

{% include modules/remember.liquid title="Tip" inline=true list=page.notes.messages %}

Suppose if we want to filter 'license' and 'publisher' fields of the 'article' store, we can index both fields into a composite index by specifying keyPath into array as follow:

    var schema = {
      store: [{
        name: 'article',
        indexes: [{
          name: 'license, publisher',
          keyPath: ['license', 'publisher']
        }]
      }]
    };

The index value is array of two elements having first element as 'publisher' field value and second element as 'license' field value of the record. The key comparison algorithm ensure that  records are sorted by license followed by publisher by iterating this index value. If we filter 'license' to 'SA', we get the result sorted by publisher as follow:
    
    key_range = ydn.db.KeyRange.starts(['SA']);
    db.values(new ydn.db.IndexValueIterator('article', 'license, publisher', key_range), 10).done(function(articles) {
        console.log(articles); 
      }
    );
        
It is notice that to get 'publisher' field sorted, we keep 'license' field constant. However we can filter 'publisher' by range.      
  
    key_range = ydn.db.KeyRange.bound(['SA', 'A'], ['SA', 'B']);
    db.values(new ydn.db.IndexValueIterator('article', 'license, publisher', key_range)).done(function(articles) {
        console.log(articles); 
      }
    );
    
Only the last field can be filtered by a range in index query. Keep in mind this limitation.
 
With above index, it is not possible to filter 'publisher' and sorted by 'license'. That require another index of `['publisher', 'license']`. This limit compound index to use in ad hoc query.
 
Note that, when iterator is used, list method do not have offset. It is for performance and robustness reasons. Skipping a number of records is not fast for large offset value. At the same time, we cannot sure that, the number of records are not change during the time. Iterator save current position and hence it can resume from the last position. For paging next view, invoke the list again with the existing iterator. It will page to the next view. Position can be be persist to storage and resume on later time. To reverse the direction, use `reverse` method of the iterator.
 
Query using compound index take only logarithmic time on object store records and hence it is very fast. But it requires more CPU time on write and extra storage for indexes. Query have to be plan before for indexing. Query are limited to one range filter. Multiple sort order is supported.
     
{% endwrap %}       