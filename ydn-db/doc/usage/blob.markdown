---
layout: ydndb-article
title: "File and Blob"
introduction: "IndexedDB can store beyond JSON serializable type, including File and Blob. In fact, File storage is so successful that W3C is even considering to use IndexedDB API over FileSystem API. "
article:
  written_on: 2014-09-5
  updated_on: 2014-09-5
  order: 4
collection: ydndb-using
authors:
  - kyawtun
---

{% wrap content %}

File and Blob should be store as single records with out-of-line key as follow:

    var file_schema = {
      stores: [
        {
          name: 'file',
          type: 'TEXT'   // data type of 'file' object store key
        }
    ]};
    db = new ydn.db.Storage('file-test-1', file_schema);
    
    /**
     * @param {string} url URL of the file to be retrieved 
     * @param {Function=} callback optional callback
     */
    var saveImage = function(url, callback) {
      var xhr = new XMLHttpRequest();
      
      xhr.open("GET", url, true);
      xhr.responseType = "blob";
       
      xhr.addEventListener("load", function () {
        if (xhr.status === 200) {
          console.log("Image retrieved");
            
          var blob = xhr.response;
     
          // Put the received blob into IndexedDB
          db.put('file', blob, url).done(function(key) {
              console.log('Save to ', 'file:' + key);
              if (callback) {callback(key);}
            }
          );
        }
      }, false);
      xhr.send();
    };
    
    /**
     * @param {string} key the key of the image record
     */
    var showImage = function(key) {
      db.get('file', key).done(function(record) {
    
        // Get window.URL object
        var URL = window.URL || window.webkitURL; 
        // Create ObjectURL
        var imgURL = URL.createObjectURL(record); 
        // Set img src to ObjectURL
        
        var img = document.createElement('img');
        img.setAttribute('name', url);
        img.setAttribute('src', imgURL);
        document.body.appendChild(img);
     
        // Revoking ObjectURL
        URL.revokeObjectURL(imgURL);
      }, function(e) {
        console.log(e.message || e);
      });
    };
    
Load and save an image file   
 
    url = 'http://dev.yathit.com/images/HTML5_logo_and_wordmark.png';
    saveImage(url); 
    
Image is retrieved by
    
    showImage(url);
        
       

{% endwrap %}        