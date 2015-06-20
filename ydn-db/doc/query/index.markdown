---
layout: ydndb-section
description: "Getting started with the YDN-DB JavaScript library and fundamental database concepts"
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
  Pages in this section include the YDN-DB script and some preloaded data and utility functions.  You can follow the sample code in your browser's developer console to see in action.

---

{% wrap content %}


IndexedDB is a key-document database with secondary indexes. SQL is decidedly absent from the IndexedDB API specification due to the difficultly in defining the SQL language. However efficiently database query remains in the IndexedDB API. We write queries with its rich, but limited, vocabulary, unambiguous grammar and syntax reminescent of the SQL language. The indexedDB API exposes only what is necessary for efficiently querying, i.e., how to query over a range of indexes. Working directly with indexes may sound clumsy, but it makes sense in JavasScript where responsiveness is paramount.  The Query API in indexedDB is simple: we either query by the primary key or a secondary key from an index. Query results are always returned one record at a time.

In this section, we will describe how to query IndexedDB using the YDN-DB library. You will need to understand the [YDN-DB database schema](../setup/schema.html) section, specifically defining [indexes](../setup/index.html).

### Test data

<script src="/js/ydn-db/data-seeding.js"></script>

{% include modules/remember.liquid title="Tip" inline=true text=page.notes.console %}

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


### Key range query for filtering and pagination

One of the simplest, as well as efficient and commonly used method is [key range](../setup/key.html#keyrange) query. Key range query allow you to query a ordered set of records between optionally lower and upper bound of an indexed field of the record. It is similar to WHERE clause in SQL.

Suppose we want to query all records with 'license' field value is 'SA'. Use `values` methods as follow:

    var key_range = ydn.db.KeyRange.only('SA');
    db.values('article', 'license', key_range).then(function(record) {
        console.log(record);
      }, function(e) {
        console.error(e);
      }
    );

For querying on primary key, skip index field name, e.g: `db.values('article', key_range)`.

By default, number of results are limited to 100. Limit and offset can be specified after key range arguments as follow:

    db.values('article', 'license', key_range, 10, 5).always(function(record) {
        console.log(record);
      }
    );

Pagination can be accomplished by using limit and offset.

Results are ordered by secondary keys and then by primary keys. Here, primary key is constant of 'SA', the results are essentially ordered by primary keys.

You can reverse the ordering by:

     db.values('article', 'license', key_range, 10, 0, true).always(function(record) {
         console.log(record);
       }
     );

If we just want to know only primary of the record, use `keys` method:

    db.keys('article', 'license', key_range).always(function(record) {
        console.log(record);
      }
    );

Key range is essentially WHERE clause in SQL. See [key range construction table](../setup/key.html#keyrange) for comparison to WHERE clause.

    SELECT * FROM article WHERE publish >= 946656000000 AND publish < 978278400000;

Above SQL query is translate to key range query by:

    var kr = ydn.db.KeyRange(946656000000, 978278400000, false, true);
    db.values('article', 'publish', kr).always(function(record) {
        console.log(record);
      }
    );

### Iterators

However instead of using key range, an iterator can be used. An iterator is essentially a combination of store name, index name and key range to specify a continuous stream of records.

    var limit = 10;
    req = db.values(new ydn.db.IndexValueIterator('article', 'license', key_range), limit);
    req.done(function(records) {
        console.log(records);
      }
    );

There are two type of iterators: key iterator and value iterator. Value iterator reference to record value whereas key iterator to index key and hence avoid serialization on retrieval. For example, we can retrieve all name starting with 'M' as follow without serialization cost of the records:

    var key_range = ydn.db.KeyRange.starts('M');
    req = db.keys(new ydn.db.IndexIterator('article', 'title', key_range), 10);
    req.done(function(titles) {
        console.log(titles);
      }
    );

It should be note that, above query is not possible in key range query syntax.

### Iterations

There are situation that index-based query is not possible or we simply don't want to storage cost of index. In this case all records in the store must be enumerate to find the desire result. This is achieved by using [open method](/api/ydn/db/storage.html#open).

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
