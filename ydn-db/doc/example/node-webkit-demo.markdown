---
layout: ydndb-article
description: "Node-Webkit demo app"
class: ydndb
title: "Node-Webkit demo app"
introduction: "YDN-DB todo list demo app built by Node-Webkit HTML5 desktop app platform."
article:
  written_on: 2013-11-14
  updated_on: 2013-11-14
  order: 3
collection: ydndb-example
authors:
  - kyawtun
---

{% wrap content %}

### Node-Webkit demo app

[node-webkit](https://github.com/rogerwang/node-webkit) is an app runtime based on [Chromium](http://www.chromium.org/) and [node.js](http://nodejs.org/). You can write native apps in HTML and JavaScript with node-webkit. It also lets you call Node.js modules directly from the DOM and enables a new way of writing native applications with all Web technologies.

Checkout pre-built [binary application files](http://ydn-db-d1.storage.googleapis.com/nw-todo/YDN-DB-Todo-0.1.0.zip) for Windows and OS X.

### Loading bundled data into the database

Applications often ship with some data.  We can fetch required files from the server or we can bundle the data files together with the application.  The following code snippet shows loading JSON data from a resource file into a YDN-DB database.

     var fs = require('fs');
     var obj;
     fs.readFile('./data.json', 'utf8', function (err, data) {
       if (err) throw err;
       var json = JSON.parse(data);
       db.put('todo', json);
     });

[Source code in Github](https://github.com/yathit/node-webkit-ydn-db-todo-sample)

{% endwrap %}
