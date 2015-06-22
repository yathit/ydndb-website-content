---
layout: ydndb-article
description: "Database Key"
title: "Key"
introduction: "Designing key is an important design decision."
article:
  written_on: 2013-04-01
  updated_on: 2014-04-28
  order: 4
collection: ydndb-setup
authors:
  - kyawtun
notes:
  console:
    Pages in this sections include the YDN-DB script and some preloaded data and utility functions, so that you follow the sample code in your browser's developer console to see in action.

---

{% wrap content %}

## Database Key

An [object store](schema.html#ydn-db-store-schema) stores records which consists of key (primary key) and value. In addition to primary key, object store optionally index secondary keys. How these keys are extracted from the record value is defined in [database schema](schema.html). Keys are important for efficient querying and are carefully constructed.
    
[Valid key](http://www.w3.org/TR/IndexedDB/#key-construct) is one of `Number`, `String`, `Date` object or `Array` consisting elements of valid key. When keys of different types are compared, `Array` is greater than `String`, which is greater than `Date` and `Date` is greater than `Number`.
    
### KeyRange
    
Records can be retrieved from object stores and indexes using either keys or [key ranges](http://www.w3.org/TR/IndexedDB/#range-concept). A key range is a continuous segment of keys defined by interval by using lower and upper bound, or unbounded.    

To retrieve all keys within a certain interval, you can construct a key range as follow:  

<table class="gridtable">
  <thead>
  <tr>
    <th>Range</th>
    <th>Code</th>
  </tr>
  </thead>
  <tbody>
  <tr>
    <td>All keys ? <strong>x</strong></td>
    <td><code>ydn.db.KeyRange.upperBound(<strong>x</strong>)</code> </td>
  </tr>
  <tr>
    <td>All keys &lt; <strong>x</strong></td>
    <td><code>ydn.db.KeyRange.upperBound(<strong>x</strong>, true) </code>
    </td>
  </tr>
  <tr>
    <td>All keys ? <strong>y</strong></td>
    <td><code>ydn.db.KeyRange.lowerBound(<strong>y</strong>)</code> </td>
  </tr>
  <tr>
    <td>All keys &gt; <strong>y</strong></td>
    <td><code>ydn.db.KeyRange.lowerBound(<strong>y</strong>, true)</code></td>
  </tr>
  <tr>
    <td>All keys &ge; <strong>x</strong> &le; <strong>y</strong></td>
    <td><code>ydn.db.KeyRange.bound(<strong>x</strong>, <strong>y</strong>)</code></td>
  </tr>
  <tr>
    <td>All keys &gt; <strong>x</strong> &lt; <strong>y</strong></td>
    <td><code>ydn.db.KeyRange.bound(<strong>x</strong>, <strong>y</strong>,
      true, true)</code></td>
  </tr>
  <tr>
    <td>All keys &gt; <strong>x</strong> &ge; <strong>y</strong></td>
    <td><code>ydn.db.KeyRange.bound(<strong>x</strong>, <strong>y</strong>,
      true, false)</code></td>
  </tr>
  <tr>
    <td>All keys &le; <strong>x</strong> &lt; <strong>y</strong></td>
    <td><code>ydn.db.KeyRange.bound(<strong>x</strong>, <strong>y</strong>,
      false, true)</code></td>
  </tr>
  <tr>
    <td>The key = <strong>z</strong></td>
    <td><code>ydn.db.KeyRange.only(<strong>z</strong>)</code></td>
  </tr>
  <tr>
    <td>All (string or array) keys start with <strong>a</strong></td>
    <td><code>ydn.db.KeyRange.starts(<strong>a</strong>)</code></td>
  </tr>
  </tbody>
</table>

For example, the key range starts with `[123]` comprises all keys of Array value with first element is `123`. For example `[123]`, `[123, -1]`, `[123, 'a']`, etc are part of the key range.

Curious reader will notice key range starts with integer has special interest. For example, `ydn.db.KeyRange.starts(2)` comprises not only 2 but a range bounded by `2 - Number.EPSILON` and `2 + Number.EPSILON`.  
  
## Nested set model

The [nested set model](http://en.wikipedia.org/wiki/Nested_set_model) is a particular technique for representing nested sets (also known as trees or hierarchies) in relational databases. Similarly Google appengine has [hierarchy key structure](https://developers.google.com/appengine/docs/python/datastore/keyclass) and AWS DynamoDB has [local secondary index](http://aws.amazon.com/blogs/aws/local-secondary-indexes-for-amazon-dynamodb/) key model. Key is constructed in such a way that parent key is in prefix resulting querying them becomes inexpensive. Parent-child and one-to-many relationship can be represent by such key structure.
 
For example, to store product in category:

    var computer_123 = {
      id: 123,
      category: 'hardware/electronic/computer/laptop/'
    };
    
We can query all computers by:
    
    var key_range = ydn.db.KeyRange.starts('hardware/electronic/computer/');
    db.values('product', 'category', key_range);
    
YDN-DB provides [Key model](/api/ydn/db/key.html) that support hierarchical key structure.
    

Key of `Array` type are useful for creating such hierarchical key structure. 
  
Suppose we have `Address` object store, which belong to `Employee` object store,
 
    var employee_123 = {
      id: 123,
      name: 'Foo Bar'
    };
    var address_123_1 = {
      id: [employee_123.id, 1]
      street: 'Street 1'
    }
    
Notice that we can efficiently retrieve all addresses belong to employee by following _key range_ query:
  
    var key_range = ydn.db.KeyRange.starts([employee_123.id]);
    db.values('Address', key_range);

{% endwrap %} 