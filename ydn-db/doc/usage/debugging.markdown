---
layout: ydndb-article
description: "Debugging ydn javascript library"
class: ydndb
title: Debugging
introduction: "We have dedicated nearly 50% of the source codes for debug information. Develop with pleasure! "
article:
  written_on: 2014-09-5
  updated_on: 2014-09-5
  order: 9
collection: ydndb-using
authors:
  - kyawtun
---

{% wrap content %}

With series of asynchronous callbacks, debugging IndexedDB or WebSQL is hard. We have dedicated nearly 50% of the source codes for debug information. These debug information are stripped on production version. To get these information, you can either use _raw version_ or _dev version_. Dev version file is postfixed with `-dev`. Raw version file is postfixed with `-raw`. If you use _dev version_, you should [turn on source map](http://www.html5rocks.com/en/tutorials/developertools/sourcemaps/) to get raw source codes, instead of minified one.

Use build-in logging facility to detail logging.

    var module = 'ydn.db';
    var level = 'finer'; // warning, info, fine, finest, all
    ydn.debug.log(module, level);
    
For more detail, see [closure library debugging facility](http://www.safaribooksonline.com/library/view/closure-the-definitive/9781449381882/ch10.html).    
    
Example output will be   
 
    [6.164s] [ydn.db.tr.Serial] B0T2 complete              
    [6.164s] [ydn.db.tr.Serial] B0T3 BEGIN ["mashCacheAge"] readonly 
    [6.165s] [ydn.db.Request] Request:sql[B0T3R2*] BEGIN    
    [6.165s] [ydn.db.sql.req.nosql.Node] sql[B0T3R2*] executing onmashCacheAge:id [MashupExamples_example3items, MashupExamples_example3items]                                 
    [6.165s] [ydn.db.sql.req.IndexedDb] sql[B0T3R2*] 4 mashCacheAge{"upperOpen":false,"lowerOpen":false "upper":"MashupExamples_example3items","lower":"MashupExamples_example3items"}
    [6.166s] [ydn.db.Request] Request:sql[B0T3R2*] END
    [6.166s] [ydn.db.Request] Request:sql[B0T3R2*] SUCCESS
     
Notice that transaction and database request print out their sequence number, eg: `B0T3R2`. Where `B` stand for branch number, `T` stands for transaction number, which is increased by one for each transaction created on the branch and `R` stands for request number, which is increased by one for each request on the transaction. `*` indicate transaction is active at the time out logging.
       
{% endwrap %}        