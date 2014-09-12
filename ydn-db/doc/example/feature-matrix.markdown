---
layout: ydndb-article
description: "NoSQL query"
class: ydndb
title: "AngularJS app on Sync"
introduction: "Very simple example for using YDN-DB library."
article:
  written_on: 2013-04-01
  updated_on: 2014-04-28
  order: 4
collection: ydndb-example
authors:
  - kyawtun
---

{% wrap content %}
 
### Feature matrix

This Angular app demonstrates how to use synchronization. [Angular](https://angularjs.org/) is one of the popular framework for developing modern web app.

Take a look at [services.js](/demo/feature-matrix/js/services.js) for how to create database service. The database service is very simple, it simple returns the storage instance. The trick with asynchronious service is that when it is consumed on [controller](/demo/feature-matrix/js/controllers.js), `$scope.$apply();` is to be invoked to activate rendering view.

Google Cloud Storage (GCS) backend server is used to synchronized data with client side database. GCS backend is essentially a RESTFul API. Only new records are delivered to the client by using monotonic increasing key.

[Feature matrix Angular.js app](/demo/feature-matrix/index.html) ([source code repo](https://github.com/yathit/feature-matrix))

{% endwrap %}     