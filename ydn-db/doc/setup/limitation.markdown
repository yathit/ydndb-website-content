---
layout: ydndb-article
description: "Supported browsers and limitation"
class: ydndb
title: Supported browsers and their limitation
introduction: "Support all desktop, mobile and web client browsers utilizing IndexedDB, WebSQL and localStorage. No dependency."
article:
  written_on: 2014-05-18
  updated_on: 2014-05-18
  order: 7
collection: ydndb-setup
authors:
  - kyawtun

---

{% wrap content %}

 * Chrome 4+ (IndexedDB or WebSql)
 * Firefox 3+ (IndexedDB draft), Firefox 10+ (IndexedDB)
 * IE 6 (userdata), IE7+ (localStorage), IE10+ desktop/mobile (IndexedDB)
 * Safari 3.1+ desktop/mobile/iOS web client (WebSql)
 * Android web client, Android browser 2.1+ (WebSql), 4+ (IndexedDB)</li>
 * Opera 10+ (WebSql), Opera 15+ (IndexedDB)
    
### Fully compatible browsers

 * Chrome (both mobile and desktop)
 * Firefox (both mobile and desktop)
    
These browsers support all features of the library and are well tested. Version of the browser is not specified since they auto-update continuously. Chrome currently does not support blob, but the feature is coming.

Older version of these browser as supported as well including Firefox 3.5 and old chrome mobile browsers. But some features (like compound index, multiEntry) may not work.

### WebSQL support

 * Chrome
 * Safari (desktop and iOS) 
 * Opera
 * iOS UIWebView
 * Cordova WebView
 * Android WebView (pre 4.3)
 * Blackberry


WebSQL supports all features of the library. The limitation are data lost on schema changes, JSON serialization instead of [structured cloning](https://developer.mozilla.org/en-US/docs/Web/Guide/API/DOM/The_structured_clone_algorithm), some database error messages and aborting.

Cursor iteration is significantly slower in WebSQL. It is suggest to use `executeSQL` or CRUD method instead, which execute natively in WebSQL.

### Modern Internet Explorer (IE11)

Internet Explorer does support IndexedDB, but not yet fully compliant as of today (May 18, 2014). Missing features are compound index, multiEntry and some schema reflection API.

### Internet Explorer 7, 8, 9

These Internet Explorer browsers does not have database, but instead use WebStorage. It is unsorted key-value store. The library use in-memory [AVL tree](http://en.wikipedia.org/wiki/AVL_tree) for query. Storage size is limited to 5MB.

Most of the features are supported.

### Internet Explorer 6

Use 1MB [User Data](http://msdn.microsoft.com/en-us/library/ms531424(v=vs.85).aspx) storage mechanism.

Most of the features are supported.

### Chrome

Prior to version 38, Chrome browser does not support serializing blob.

## Incomplete implementation

### Event module

Event module implementation is incomplete. See [release note](http://dev.yathit.com/ydn-db/ydn-db-version-1-release.html) for more detail.

{% endwrap %} 