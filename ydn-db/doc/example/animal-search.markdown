---
layout: ydndb-article
description: "YDN-DB example demo application and unit tests"
class: ydndb
title: "Full-text search"
introduction: "List of YDN-DB example applications and unit test suites."
article:
  written_on: 2014-04-28
  updated_on: 2015-07-26
  order: 5
collection: ydndb-example
authors:
  - kyawtun
---
{% wrap content %}

### Animal search

This animal serach app show how to use full-text search. Input text field is suggested by a [datalist](https://developer.mozilla.org/en/docs/Web/HTML/Element/datalist) as user type in the input field.

[Animal database search](http://yathit.github.io/ydndb-demo/ydn-db-text/animals/animals.html)


### Opportunistic caching

Pubmed search app demonstrate free text search query and render query result. The search result token are highlighted on the original document.

The app also show how to utilize opportunistic caching, in which, query are perform first on the client and then, if satisfactory result is not found, query again on backend server and cached on the client side.

[Pubmed search app](http://yathit.github.io/ydndb-demo/ydn-db-text/pubmed-search/index.html)
  ([source code repo](https://github.com/yathit/ydn-db-fulltext))


{% endwrap %}
