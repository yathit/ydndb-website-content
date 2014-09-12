---
layout: ydndb-article
description: "Debugging ydn javascript library"
class: ydndb
title: Debugging
introduction: "Develop with pleasure"
article:
  written_on: 2014-09-5
  updated_on: 2014-09-5
  order: 9
collection: ydndb-using
authors:
  - kyawtun
---

{% wrap content %}

Use dev version of the compile JS file during development. Dev version are postfix with `-dev` on library js file anme. Put .map file in the same directory as js file. Turn on source map for debugging with full source code. Use build-in logging facility to detail logging.

    var module = 'ydn.db';
    var level = 'finer'; // warning, info, fine, finest, all
    ydn.debug.log(module, level);
    
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