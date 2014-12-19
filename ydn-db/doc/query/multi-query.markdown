---
layout: ydndb-article
description: "Multi query in indexeddb"
class: ydndb
title: Multi query
introduction: "Union of multiple query results."
article:
  written_on: 2014-12-18
  updated_on: 2014-12-18
  order: 4
collection: ydndb-query
authors:
  - kyawtun
notes:
  console:
    Pages in this sections include the YDN-DB script, so that you follow the sample code in your browser's developer console to see in action.

---

{% wrap content %}

YDN-DB provides method for running multiple query in parallel, allowing for executing `IN` query in SQL syntax.

Consider the following query, `SELECT * FROM article WHERE topics IN ("object-relational mapper", "open source") LIMIT 10`

Create iterator each for all element of IN condition clause.

    var iter_0 = ydn.db.IndexIterator.where('article', 'topics', '=', 'object-relational mapper');
    var iter_1 = ydn.db.IndexIterator.where('article', 'topics', '=', 'open source');

Create multi query function, which keep resulting primary keys and advance cursor(s) on each cursor iterator in such a way that, cursor(s) is advance for next primary key.

    var keys = [];
    var multiQuery = function(sec_keys, pri_keys) {
      var advance = [];
      var cmp = ydn.db.cmp(pri_keys[0], pri_keys[1]);
      if (cmp == 1) { // pri_keys[0] > pri_keys[1]
        if (keys[keys.length - 1] != pri_keys[1]) {
          keys.push(pri_keys[1]);
        }
        advance[1] = true; // advance iter_1 on step
      } else if (cmp == -1) { // pri_keys[0] < pri_keys[1]
        if (keys[keys.length - 1] != pri_keys[0]) {
          keys.push(pri_keys[0]);
        }
        advance[1] = true; // advance iter_1 on step
      } else { // pri_keys[0] == pri_keys[1]
        if (keys[keys.length - 1] != pri_keys[0]) {
          keys.push(pri_keys[0]);
        }
        advance[0] = true; // advance iter_0 on step
        advance[1] = true; // advance iter_1 on step
      }

      if (keys.length >= 10) {
        return [];
      } else {
        return advance;
      }
    };

Open database if not done so. You must load the data as described in [overview](index.html) section.

    var db = new ydn.db.Storage('nosql-query');

Here, scanning query is performed using `scan` method. Notice that after key scanning, function record value are retrieve by `values` method.

    db.scan(multiQuery, [iter_0, iter_1]).done(function() {
      db.values('article', keys).done(function(values) {
        console.log(keys, values);
      })
    });


{% include modules/remember.liquid title="Tip" inline=true text=page.notes.console %}

We can resume the query from the previous by issue the query again. It is possible to reverse the query by changing the sing inside `multiQuery` function. These are patterns for pagination in ydn-db query, which does not use offset.

`scan` method is very general and can be combine with [key joining algorithm](key-joining.html) described previously. It should be notice that multi-query is [logical disjunction, OR](http://en.wikipedia.org/wiki/Logical_disjunction), while key joining is [logical conjunction, AND](http://en.wikipedia.org/wiki/Logical_conjunction).

[Negation, NOT](http://en.wikipedia.org/wiki/Negation) is implemented by multiple query of open lower bound key range and open upper bound key range query. For example, the query `SELECT * FROM article WHERE topics != "open source"` is result of multi query of iterators `ydn.db.IndexIterator.where('article', 'topics', '>', 'open source')` and `ydn.db.IndexIterator.where('article', 'topics', '<', 'open source')`.

By re-writing query into [canonical normal form](http://en.wikipedia.org/wiki/Canonical_normal_form),  combination of these queries can be executed. There are no limit in number of query, but only one range query can be in the combination. For more detail about these query see Alfred Fuller's [Next gen queries](https://www.youtube.com/watch?v=ofhEyDBpngM) in Google I/0 2010.

{% endwrap %}
