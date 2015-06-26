---
layout: ydndb-article
description: "Encryption examples using YDN-DB library"
class: ydndb
title: "Encryption"
introduction: "Encryption examples using YDN-DB library."
article:
  written_on: 2015-07-26
  updated_on: 2015-07-26
  order: 9
collection: ydndb-example  
authors:
  - kyawtun
---

{% wrap content %}
 
### Todo list

[Todo list app](https://yathit.github.io/ydndb-demo/encrypt/todo.html) is a very simple app to demonstrate basic usage of encryption in ydn-db. It show how to put data and get out data from the database. It is based on [the todo list example](http://www.html5rocks.com/en/tutorials/indexeddb/todo/) in HTML5Rocks web site.

### Crypto module

Crypto module can be used outside of ydn-db, if record key is not necessary to be encrypted. Using crypto module outside of ydn-db provides more flexible usage. 

[Crypto example](https://yathit.github.io/ydndb-demo/encrypt/crypto-example.html) demonstrates a simple use case for crypto module with ydn-db.

[Crypto with index example](https://yathit.github.io/ydndb-demo/encrypt/crypto-with-index.html) demonstrates query meta data on encrypted record. Note meta data for index are expose of encrypted message.
   
{% endwrap %}     