---
layout: ydndb-article
description: "How to install YDN-DB library in your app"
title: Setup
introduction: "YDN-DB is a pure javascript library, and hence include the library javascript file to your HTML page to use it. But there are elaborate setup in large scale web app development."
article:
  written_on: 2013-04-01
  updated_on: 2014-04-28
  order: 2
collection: ydndb-setup
authors:
  - kyawtun
notes:
  shaky:
    Currently ydn-db repository in GitHub does not commit minified files breaking npm or bower. :-(

---

{% wrap content %}

## Loading YDN-DB javascript library


The namespace of this library is <code>ydn.db</code>. The script provide main javascript class object, called <code>ydn.db.Storage</code>.

### In browser

To use YDN-DB library, simple include the library file before using in your HTML file, as follow:

    <script type="text/javascript" src="ydn.db-iswu-core-e-qry-dev.js"&gt;&lt;/script>      
    
### With AMD loader

The pre-build minified expose as both [AMD](http://requirejs.org/docs/whyamd.html) and [CommonJS](http://wiki.commonjs.org/wiki/CommonJS) compatible module format.

    require({
      'packages': [{'name': 'ydn', 'location': 'path/to/ydn-db', 'main': 'ydn.db-iswu-core-e-qry-dev'}]
      }, ['ydn'], function(ydn) {
      console.log(ydn.db.version);
      var db = new ydn.db.Storage('db name');
    });
    
### With CommonJS loader or browserify
    
    var ydn = ydn || {};
    ydn.db = require('ydn.db');
    
  
### Bower
  
Installed by <a href="http://bower.io">bower</a>:

    bower install ydn.db   
    
{% include modules/remember.liquid title="Caution" inline=true text=page.notes.shaky %}

### Using in PhoneGap (Cordova)

PhoneGap is an open source framework that allows you to create mobile apps using standardized web APIs, specifically [Apache Cordova](http://cordova.apache.org/) API. The app is rendered in web client view such as [UIWebView](https://developer.apple.com/library/ios/documentation/uikit/reference/UIWebView_Class/Reference/Reference.html) in iOS and [WebView](http://developer.android.com/reference/android/webkit/WebView.html). The usage of YDN-DB in these mobile client are same as in browser. But more difficult to use then in browser because of limited debugging capability, more restriction and wide different environments.
 
See [iOS 7 todo example](../example/ios-7.html) for developing in iOS. Before iOS 8, only WebSQL is available. IndexedDB is available in iOS 8 mobile client.
 
See [Cordova todo example](../example/cordova-2.3.html) for developing in Android. WebSQL is available in Android. IndexedDB is only available after Android 4.2, but buggy and advice to use WebSQL instead. IndexedDB support on Android 4.4 is good and recommended.
  
## Supported database storage engines
  
YDN-DB use database storage mechanism independent abstraction layer, although it is designed primarily for IndexedDB storage mechanism. Fortunately WebSQL and key-value store (WebStorage) can be polyfill without serious issue. key-value store engines supports `localStorage`, `sessionStorage`, `UserData` (IE6) and in-memory storage mechanisms.
  
When using pre-build files in [download page](/ydn-db/downloads.html), you can choose to support multiple storage mechanism. All pre-build files has symbol representing supported storage mechanism by their first letter. For example `isw` symbol denotes supporting for IndexedDB (i), WebSQL (s) and WebStorage (w).    
   
## YDN-DB modules
   
YDN-DB source code are composed of modules. Currently there are nine modules available in pre-build files in [download page](/ydn-db/downloads.html).

|---
| Name | Symbol | Typical use case | Description |
|-|-|-|
| Core | core | `ydn.db.Iterator` | Core query |
| CRUD | crud | `db.get()`, `db.put()` | Basic CRUD query |
| Cursor | cur | `db.open()`, `db.scan()` | Cursor iteration |
| Encryption | ept | `schema.Encryption` | Encryption |
| Event | e | `db.addEventListener()` | Database events |
| Full-text search | text | `db.search()` | Full-text search |
| Query | qry | `db.from()` | Query |
| SQL | sql | `db.executeSql()` | SQL |
| Synchronization | sync | `schema.Sync` | Synchronization |

When using pre-build files in [download page](/ydn-db/downloads.html), you can choose to various combination of modules. For example, the pre-build file, 'ydn.db-isw-core-qry.js' contains core module and query module.

If desire combination of module are not available in download page, you can request special build for paid user. Custom build script are provided for paid user.

{% endwrap %}    
