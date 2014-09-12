---
layout: ydndb-article
description: "NoSQL query"
class: ydndb
title: "Sqliteplugin on Phonegap"
introduction: "Very simple example for using YDN-DB library."
article:
  written_on: 2013-04-01
  updated_on: 2014-04-28
  order: 3
collection: ydndb-example
authors:
  - kyawtun
---

{% wrap content %}
 
### Todo list (Sqliteplugin on Phonegap)

WebSQL storage size is limited to 50 MB and sometimes its durability is under question. [Sqliteplugin](https://github.com/brodysoft/Cordova-SQLitePlugin) for Cordova, however use bundled sqlite, instead of WebSQL on WebView offering larger storage size and better durability. It is also possible to use some SQL statement not supported in WebSQL.

The instruction on the project repo show all instruction necessary to setup and run on iOS and Android.

[Phonegap app](https://github.com/yathit/cordova-sqliteplugin-todo) for iOS and Android with [Sqliteplugin](https://github.com/brodysoft/Cordova-SQLitePlugin) (Cordova 3.4.0)
   
{% endwrap %}     