---
layout: ydndb-section
description: "Getting started with YDN-DB javascript library and fundamental database concepts"
class: ydndb
title: Query
introduction: "YDN-DB library query in-depth."
article:
  written_on: 2013-04-01
  updated_on: 2014-04-28
  order: 2
collection: ydndb-doc
id: ydndb-query
notes:
  console:
    Pages in this sections include the YDN-DB script and some preloaded data and utility functions, so that you follow the sample code in your browser's developer console to see in action.

---

{% wrap content %}

  
IndexedDB is a key-document database and does not have a SQL processor. Query in IndexedDB is scanning keys over a range. For more complex query, application is responsible to join or place in buffer to get necessary result. SQL processor performs the same thing in the lower level during its query execution. 

In javascript use case, such low level key scanning gives perceive fast response since we are getting immediate result, record-by-record. Whereas in WebSQL, all processing are performed in the database and return to javascript only after all finished. Consequently, memory and CPU times may have been wasted if the result are no longer require by the UI process.  

### Test data

{% include modules/ydndb_data_js.liquid %}

Before we start, let us populate the database with some data base on previous schema. Schema definition and random data generation are defined in [data-seeding.js](http://dev.yathit.com/js/ydn-db/data-seeding.js) file.
        
    db = new ydn.db.Storage('nosql-query', author_article_topic_schema);
    
    db.count('topic').done(function(cnt_topic) {
      var n_topics = 10;
      var n_authors = 100;
      var article_per_author = 10;
      if (cnt_topic < n_topics) {
        var topics = topics = genTopics(n_topics);
        db.put('topic', topics).done(function(t_keys) {
          console.log(t_keys.length + ' topics added.');
          var authors = genAuthors(n_authors);
          var emails = authors.map(function(x) {return x.email;});   
          var articles = genArticles(article_per_author*n_authors, t_keys);
          var article_keys = articles.map(function(x) {return pickOne(emails) + '/' + x.publish;});
          db.put('author', authors).done(function(x) {console.log(x.length + ' authors added.');});
          db.put('article', articles, article_keys).done(function(x) {console.log(x.length + ' articles added.');});
        });
      } else {
        console.log('db ready.');
      }
    });   
         
Above code snippet will generate three stores and randomly populate them with 10 topics, 100 authors and 1000 articles. An example of author record and article records is as follow: 
        
    author_1 = {
      email: 'me@aaronsw.com',
      born: 531763200000,
      first: 'Aaron',
      last: 'Swartz',
      company: 'Reddit',
      hobby: ['programming', 'blogging', 'politics']
    };
    article_1 = {
      title:"Database process integrated library system into a class",
      content:"...",
      license:"NC",
      publisher:"Springer",
      publish:640022400000,
      topics:["object-relational mapper","website wireframe","distributed computing","dynamic systems development method","open source","integrated library system"]}    
           
Article store use out-off-line primary key as composite key of parent key, author email, and publish date. Ideally composite key will use array key, but here we just concat them so that it is compatible with IE11, which does not support array key.   
        
### Filtering 
       
Filtering or retrieving records in a key range of indexed files is as efficient as primary key query. The query field must be indexed. In this library, it is performed as follow: 
      
    var key_range = ydn.db.KeyRange.only('SA');
    req = db.get(new ydn.db.IndexValueIterator('article', 'license', key_range));
    req.then(function(record) {
        console.log(record);
      }, function(e) {
        throw e;
      }
    );
          
However instead of giving ydn.db.KeyRange, to get method, an iterator is used. This is to avoid confusion and high light the fact that we are iterating. Unlike primary key, secondary key can reference to multiple records. If multiple results are expected, list method can be use as follow:    
      
    var limit = 10;
    req = db.values(new ydn.db.IndexValueIterator('article', 'license', key_range), limit);
    req.done(function(records) {
        console.log(records);
      }
    );
          
An iterator is essentially a combination of store name, index name and key range to specify a continuous stream of records. There are two type of iterators: key iterator and value iterator. Value iterator reference to record value whereas key iterator to index key and hence avoid serialization on retrieval. For example, we can retrieve all name starting with 'M' as follow without serialization cost of the records: 
         
    var key_range = ydn.db.KeyRange.starts('M');
    req = db.values(new ydn.db.IndexIterator('article', 'title', key_range), 10);
    req.done(function(titles) {
        console.log(titles);
      }
    );
             
If the field is not indexed, all records in the store must be iterated. This is achieved by using open method.           
    var count = 0;
    req = db.open(function(cursor) {
      var record = cursor.getValue();
      console.log(record.first + ' ' + record.last);
      count++;
    }, new ydn.db.ValueIterator('author'));
    req.done(function() {
      console.log(count + ' authors found.');
    }, 'readonly');  
    
Opening can be invoked by `'readonly'` or `'readwrite'` mode.
 
In general, query involves filtering multiple fields, paging and the results are to be sorted. Such complex queries are solved by using index, [compound index](compound-index.html) and algorithmic [key joining](key-joining.html).    

{% endwrap %}    