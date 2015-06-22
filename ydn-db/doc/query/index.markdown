---
layout: ydndb-section
description: "YDN-DB library query in-depth."
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

Before starting, you should populate the database with some data based upon previous schema. Schema definition and random data generation are defined in the [data-seeding.js](http://dev.yathit.com/js/ydn-db/data-seeding.js) file.

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

The above code snippet generates three stores and randomly populates them with 10 topics, 100 authors and 1000 articles.  Here is an example of an author record and an article record:

    author_1 = {
      email: 'me@aaronsw.com',
      born: 531763200000,
      first: 'Aaron',
      last: 'Swartz',
      company: 'Reddit',
      hobby: ['programming', 'blogging', 'politics']
    };
    article_1 = {
      title:"Database Processes for an Integrated Library System in a Class",
      content:"...",
      license:"NC",
      publisher:"Springer",
      publish:640022400000,
      topics:["object-relational mapper","website wireframe","distributed computing","dynamic systems development method","open source","integrated library system"]}

The article store uses an out-of-line primary key which is a composite key composed of the parent key, the author's email, and the publication date (publish field).  While a composite key is, ideally, an array of the composed keys, this just concatenates the fields to maintain compatibility with IE11. IE11 does not support array keys.


### Key range query for filtering and pagination

One of the simplest, efficient, and common filter methods is the [key range](../setup/key.html#keyrange) query. A key range query allows you to query a ordered set of records, optionally specifying a lower and upper bound of an indexed field. It is similar to WHERE clause in SQL.

To query all records with the 'license' field value 'SA', use the `values` method as follows:

    var key_range = ydn.db.KeyRange.only('SA');
    db.values('article', 'license', key_range).then(function(record) {
        console.log(record);
      }, function(e) {
        console.error(e);
      }
    );

To query on the primary key, skip the index field name, e.g: `db.values('article', key_range)`.

By default, the number of results is limited to 100. Limit and offset arguments can be specified after the key range arguments:

    db.values('article', 'license', key_range, 10, 5).always(function(record) {
        console.log(record);
      }
    );

Pagination can be accomplished by using limit and offset.

Results are ordered by secondary keys and then by primary keys.  In the above example, the primary key is a constant of 'SA', so the results are ordered by the secondary key.

You can reverse the ordering with a boolean:

     db.values('article', 'license', key_range, 10, 0, true).always(function(record) {
         console.log(record);
       }
     );

If we just want to know the primary keys of records, use the `keys` method:

    db.keys('article', 'license', key_range).always(function(record) {
        console.log(record);
      }
    );

The key range is essentially the WHERE clause in SQL. See [key range construction table](../setup/key.html#keyrange) for a comparison to the WHERE clause.

    SELECT * FROM article WHERE publish >= 946656000000 AND publish < 978278400000;

The above SQL query is translated to a key range query as:

    var kr = ydn.db.KeyRange(946656000000, 978278400000, false, true);
    db.values('article', 'publish', kr).always(function(record) {
        console.log(record);
      }
    );

### Iterators

An iterator can be used instead of a key range. An iterator specifies a continuous stream of records by combining a store name, index name and key range.

    var limit = 10;
    req = db.values(new ydn.db.IndexValueIterator('article', 'license', key_range), limit);
    req.done(function(records) {
        console.log(records);
      }
    );

There are two type of iterators: key iterators and value iterators.  A value iterator references values in a record and requires serializaiton on retrieval.  A key iterator references only the indexed key and avoids the serializaiton cost. For example, we can retrieve all names starting with 'M' without the serialization cost of the records:

    var key_range = ydn.db.KeyRange.starts('M');
    req = db.keys(new ydn.db.IndexIterator('article', 'title', key_range), 10);
    req.done(function(titles) {
        console.log(titles);
      }
    );

Note that the above query is not possible using the key range query syntax.

### Iterations

There are situations that index-based queries are not possible or you simply don't want the storage cost of an index. In these cases, all records in the store must be enumerated to find the desired result using the [open method](/api/ydn/db/storage.html#open):

    var count = 0;
    req = db.open(function(cursor) {
      var record = cursor.getValue();
      console.log(record.first + ' ' + record.last);
      count++;
    }, new ydn.db.ValueIterator('author'));
    req.done(function() {
      console.log(count + ' authors found.');
    }, 'readonly');

Opening can be invoked in `'readonly'` or `'readwrite'` mode.

In general, queries involves filtering multiple fields, paging results, and choosing how results will be sorted. Complex queries are solved by using index, [compound index](compound-index.html) and algorithmic [key joining](key-joining.html) techniques.

{% endwrap %}
