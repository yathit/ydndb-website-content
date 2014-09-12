---
layout: ydndb-article
description: "NoSQL query"
class: ydndb
title: Key joining
introduction: "Key joining is not build into IndexedDB API."
article:
  written_on: 2013-04-01
  updated_on: 2014-04-28
  order: 3
collection: ydndb-query
authors:
  - kyawtun
notes:
  console:
    Pages in this sections include the YDN-DB script and some preloaded data and utility functions, so that you follow the sample code in your browser's developer console to see in action.
g_comments_href:
  http://dev.yathit.com/ydn-db/nosql-query.html
---

{% wrap content %}

Filtering is, in fact, joining on primary key. The join process will scan index keys of desire filter and the matching primary keys are result. Numerous join algorithms can be found in [database books](#references) and review articles.
 
`scan` database operation method is used for key scanning process. The method accepts array of iterators (usually key iterator for performance) and a callback for each iteration. The callback is invoked with two argument of arrays having primary keys and secondary keys (record values if value iterator is used). The callback return an advancement array. Each advancement element refer to the respective iterator, `true` to continue next iteration, false to restart the iteration. If any value is given, it is taken as primary key and advance to it within current index key range. If a primary key larger than given key is found, it was returned on the first occurrence.
 
In general, the callback can return an object having fields of advance, `next_index_keys` and `next_primary_keys`. All three fields are arrays, with each element represent to respective iterator. If element of next_index_keys is a valid key, the cursor advance to it. If element of `next_primary_keys` is given, the cursor advance to it within given index key or current index key. If element of advance array is given, it must be boolean. true refer to next position and false to rewind the iterator. If all three are given, it starts with rewind, positioning and advancement.
 
Available algorithms can be found in [`ydn.db.algo`](/api/ydn/db/algo.html) module.

### Nested-loop join

A naive join algorithm iterates each iterator forming nested loops. And hence it is called nested-loop join algorithm.
  
The following snippet filter 'license' field to 'SA' and 'publisher' to 'Nature'.

    match_keys = [];
    var nested_loop = function(keys, values) {
      // here: keys = index keys, values = primary keys
      // index 0 refer to license field and index 1 to publisher
      if (keys[0] != null) {
        if (values[1] != null && ydn.db.cmp(values[0], values[1]) == 0) {
          match_keys.push(values[0]); // we got the matching primary key.
        } 
        if (keys[1] != null) {
          return [null, true]; // iterate on publisher
        } else {
          return [true, false]; // iterate on license and restart on publisher
        }
      } else {
        return []; // no more iteration. we are done.
      }
    };
    
    license_filter = new ydn.db.IndexIterator('article', 'license', ydn.db.KeyRange.only('SA'));
    publisher_filter = new ydn.db.IndexIterator('article', 'publisher', ydn.db.KeyRange.only('Nature'));
    var req = db.scan(nested_loop, [license_filter, publisher_filter]);
    req.then(function() {
      db.values('article', match_keys).done(function(results) {
        console.log(results);
      });
    }, function(e) {
      throw e;
    });
    
The inner loop, publisher, iterate the number of licenses times. Whereas the outer loop, license iterate only once. The result is the same if we flip the loop order since the joins are commutative. However run time is not the same. This is the subject of query optimization.
  
The inner loop result can be cached into the memory. This lead to hash merge algorithm.  
  
### Sorted-merge join
  
In the above example, we noticed that the results of both iterators are sorted by ascending order of primary key since we held constant on index key. If the results of all filters are sorted, we can advance the iterator skipping keys not in the filter. This make sorted join algorithm independent of number of records in the object store.   

    match_keys = [];
    var sorted_join = function(keys, values) {
      if (keys[0] != null && keys[1] != null) {
        var cmp = ydn.db.cmp(values[0], values[1]);
        if (cmp === 1) {
          return {continuePrimary: [null, values[0]]}; // move publisher to match license
        } else if (cmp === -1) {
          return {continuePrimary: [values[1], null]}; // move license to match publisher
        } else {
          match_keys.push(values[0]); // we got the matching primary key.
          return [true, true]; // move forward both
        } 
      } else {
        return []; // no more iteration. we are done.
      }
    };
    
    license_filter.reset();
    publisher_filter.reset();
    var req = db.scan(sorted_join, [license_filter, publisher_filter]);
    req.then(function() {
      db.values('article', match_keys).done(function(results) {
        console.log(results);
      });
    }, function(e) {
      throw e;
    });
    
Notice that sorted merge out perform nested loop in order magnitude. Using naive nested loop join in a practical web app is catastrophic.
 
 We can limit the result by existing the loop any by returning empty array to the iteration callback. Iterators are resume if use again. That is why we need to reset the iterators since they haven been used in nested loop. This how paging is handle in scanning.
 
 The result of sorted merge join is sorted by primary key. To sort by a specific column field other than primary key, a zigzag merge join algorithm is used. 
    
### Zigzag merge join
      
In sorted merge join, effective key is held constant while joining sorted list of values. In Zigzag merge join, the effective itself comprise both parts. The first part, known as pre-fix is held constant consisting what we want to be filtered. The second parts, known as post-fix is sorted value. The composite index is used to construct post-fix and prefix as required by the query. 

To query `SELECT * FROM article WHERE license = 'SA' AND publisher = 'Nature' ORDER BY 'title'`, we construct two iterator using composite index of `['license', 'title']` and `['publisher', 'title']`. 'title' is the postfix we are joining. The 'license' prefix is held constant to 'SA' and 'publisher' prefix is held constant to 'Nature' as follow.   
 
    var license_sa_iter = ydn.db.IndexIterator.where('article', 'license, title', '^', ['SA']);
    var publisher_nature_iter = ydn.db.IndexIterator.where('article', 'publisher, title', '^', ['Nature']);
    var match_keys = [];
    var solver = new ydn.db.algo.ZigzagMerge(match_keys);
    var req = db.scan(solver, [license_sa_iter, publisher_nature_iter]);
    req.then(function() {
      db.values('article', match_keys).done(function(results) {
        console.log(results);
      });
    }, function(e) {
      throw e;
    }); 
 
### References
 * Héctor García-Molina, Jeffrey D. Ullman, Jennifer Widom <a href="http://books.google.com.sg/books?id=jOVQAAAAMAAJ">Database System Implementation</a> Prentice Hall, 2000.
       
       
{% endwrap %}     