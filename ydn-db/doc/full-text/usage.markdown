---
layout: ydndb-article
description: "Full text search module for YDN-DB"
introduction: "Explain YDN-DB full-text search usage."
title: Usage
article:
  written_on: 2012-04-01
  updated_on: 2014-04-28
  order: 1
collection: ydndb-full-text
authors:
  - kyawtun

---

{% wrap content %}

### Enabling full-text support

To use full-text search feature, you must use one of the pre-build library containing [`'text'` module](../setup/setup.html#ydn-db-modules), such as 'ydn.db-isw-core-e-qry-text.js'. If right mixed of modules are not available in pre-build [download page](/ydn-db/downloads.html), custom build can be requested for paid user.

### Full-text catalogs

A full-text catalog is a logical container for full-text indexes, to make management of groups of full-text indexes easier. [Full-text catalog](/api/ydn/db/schema.html#Catalog) is defined in [YDN-DB database schema](/api/ydn/db/schema.html#Database). 

The following full text catalog index author name on `first` and `last` field
of record value with weighting more on `first`.
 
    var schema = {
      fullTextCatalogs: [{
        name: 'name',
          sources: [
            {
              storeName: 'contact',
              keyPath: 'first'
            }],
        ]},
        stores: [
          {
            name: 'contact',
            autoIncrement: true
          }]
    };
    
Each full-text (inverted) index has source reference to original document by `storeName` and `keyPath`. The value of `keyPath` is the text to be indexed. `weight` factor is applied when ranking search result. This weight value is not stored in the database can be changed after indexing as well.    
    
### Usage

Storage instantiation is same, but use schema with `fullTextCatalogs` attribute as described above. `put` and `add` method will trigger [full-text indexing process](basic.html#indexing-process). `remove` or `clear` methods will remove full-text index as well.
    
    var db = new ydn.db.Storage('db name', schema);
    db.put('contact', [{first: 'Jhon'}, {first: 'Collin'}]);
    
Full-text search is performed by using `search` method giving full-text catalog name and query terms.    
    
    db.search('name', 'jon').done(function(x) {
      console.log(x);
      db.get(x[0].storeName, x[0].primaryKey).done(function(top) {
        console.log(top);
      })
    });
    
Full-text search request return list of inverted index. An inverted index has the following attributes: `storeName`, `primaryKey`, `score`, `tokens`, representing for store name of original document, primary key of original document, match quality score and array of token objects. Token object has the following attributes: `keyPath`, `value` and `loc` representing key path of index of the original document, original word from the original document and array list of position of word in the document. 
        
### Query format
        
Query format is free text, in which implicit and/or/near logic operator apply for each token. Use double quote for exact match, - to subtract from the result and * for prefix search.        

{% endwrap %}   