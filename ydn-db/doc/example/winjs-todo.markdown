---
layout: ydndb-article
description: "YDN-DB todo list demo app built by WinJS for Windows Store App."
class: ydndb
title: "Windows Store App"
introduction: "YDN-DB todo list demo app built by WinJS for Windows Store App."
article:
  written_on: 2013-11-14
  updated_on: 2013-11-14
  order: 4
collection: ydndb-example
authors:
  - kyawtun
---

{% wrap content %}

### Windows Store App

[Windows Store App](https://dev.windows.com/en-us/getstarted) for using YDN-DB javascript database library on HTML5 platform. The example include loading resource data file into the database.

#### Bundling data to load into database

It is commonly required that to ship some data together with the application. We can fetch require files from the server, but to work on offline scenario, we can bundle data files together with the application. The follow code snippet shows loading JSON data from the resource file to YDN-DB database.

    var url = new Windows.Foundation.Uri("ms-appx:///data/data.json");
    Windows.Storage.StorageFile.getFileFromApplicationUriAsync(url).then(function (file) {
        Windows.Storage.FileIO.readTextAsync(file).then(function (text) {
            var json = JSON.parse(text);
            db.put('todo', json).done(function () {
                getAllTodoItems();
            });
        });
    });

[Source code in MSDN](https://code.msdn.microsoft.com/YDN-DB-javascript-database-cf28aa4d)

[Visual Studio Community 2013](http://www.visualstudio.com/products/visual-studio-community-vs) is required to build the solution.

{% endwrap %}
