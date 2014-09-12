---
layout: ydndb-article
title: "Modeling relationships"
introduction: "Data modeling with YDN-DB"
article:
  written_on: 2014-09-5
  updated_on: 2014-09-5
  order: 1
collection: ydndb-using
authors:
  - kyawtun

---

{% wrap content %}


The NoSQL style databases has often been termed non-relational databases. This is an unfortunate misconception. As we will see, IndexedDB can easily model relationships. The real problem, however, is it does not enforce integraty of the relationships.

A relationship is essentially a reference from one object to an object. Which means, making a relationship is as simple as keeping a reference of an object in an object. The reference will be a primary key. The three kinds of relationships, one-to-one, one-to-many, many-to-many are modeled by storing reference. The discussion here is how to records these reference so that bi-directional transversable, efficient and consistent. Unlike RMDB system, NoSQL do not have native support for relationship. In fact, getting ride of relationship is the main objective of NoSQL so as to offer in scalability and performance. People tern to over-rely on relationship model suffering un-necessary performance penalty.

### One-to-many relationship

One-to-many relationship is modeled by keeping 'one' reference to objects in 'many' side. Reference is simply a primary key, which could be a string or an integer.

Refering to 'author' object store in previous session, the 'author', store has one index of named 'company'. 'author' store is related to 'company' store by defining the following store schema.  

    company_store_schema = {
      name: 'company',
      keyPath: 'name',
      type: 'TEXT'
    };
    
Additionally, the value of 'company' attribute in the 'author' record is restricted to valid primary key of company records. This consistency has to be inforce by the application. In the future version, the library may add such foreign key relationship enforcement.  
  
    schema = {
      stores: [author_store_schema, company_store_schema]
    };
    
    db = new ydn.db.Storage('test-2', schema);
    db.put('company', companies);
    db.count(['author', 'company']).done(function(cnts) {
      console.log(cnts[0] + ' authors and ' + cnts[1] + ' companies.');
    });
      
We can query the relationship from both direction. To query authors on particular company:
      
    db.values('author', 'company', ydn.db.KeyRange.only('Google'), 10).done(function(authors) {
      console.log(authors);
    });
          
### One-to-one relationship
 
One to one relationships are essentially a special-case of one to many relationships. Like one to many relationships, they are usually accomplished with a reference primary key on one side referring to the other object store.
 
### Parent-child relationship
 
Parent child relationship is in face one to many relationships. To enforce, parent-child relationship, we can use array keyPath in child object store. The primary key of the child record consists primary key of parent object store and child id. The following 'article' child store schema illustrate parent-child relationship with 'author' store.        
  
    blog_schema = {
      stores: [{
        name: 'author',
        keyPath: 'email'
      }, {
        name: 'article',
        indexes: [{
          keyPath: 'title'
        }, {
          keyPath: 'tag',
          multiEntry: true
        }]
      }]
    };
      
The 'author' store use 'email' attribute for in-line primary key. Using email for primary key is smart because email is unique to author. Email is the most likely information we know and query. _Data type_ of 'TEXT' is given to primary key by using type _schema attribute_. It is not used in IndexedDB, but used in WebSQL. In fact, it is required to perform native SQL query.
 
The 'article' store do not specified keyPath schema attribute and hence it must use out-of-line key. The primary key type is specified as an array of two 'TEXT'. It uses array because it has two parts. Its first element is primary key of 'author' store ('email') and the second element is 'article' 'title'. This key construction enforces that 'article' must has one 'author' and a unique title for each author. 'author' store is called _parent store_ and 'article' store is called _child store_.

'title' of 'article' attribute is indexed and hence it becomes _index key_. Like primary key, index key can be used to retrieve the record directly. Unlike primary key, index key is not necessary to be unique. However unique integrity is desired, it can be enforced by unique schema attribute. 
 
'tag' is _listed index_ because its `multiEntry` schema attribute set to true. Listed index field value must be an array, otherwise it will not be indexed. Listed index field value are indexed element-wise. Here is sample data:
       
    author1 = {
      first: 'Micheal',
      last: 'Bolin',
      email: 'bolinfest@gmail.com'
    };
    author2 = {
    first: 'Kyle',
    last: 'Huey',
    email: 'me@kylehuey.com' 
    }
    article1 = {
      title: 'Inheritance Patterns in JavaScript',
      url: 'http://bolinfest.com/javascript/inheritance.php',
      tag: ['javascript', 'design pattern', 'closure tools']
    }
    article2 = {
      title: 'Writing useful JavaScript applications in less than half the size of jQuery',
      url: 'http://blog.bolinfest.com/2011_07_01_archive.html',
      tag: ['javascript', 'closure tools', 'jquery']
    }
    article3 = {
      title: 'Fixing the Memory Leak',
      url: 'http://blog.kylehuey.com/post/21892343371/fixing-the-memory-leak',
      tag: ['browser', 'firefox', 'memory']
    }
    article4 = {
      title: 'Pushing Compilers to the Limit (and Beyond)',
      url: 'http://blog.kylehuey.com/post/14453464655/pushing-compilers-to-the-limit-and-beyond',
      tag: ['javascript', 'firefox', 'performance', 'jit']
    }

Create a new database and insert them.
       
    blog_db = new ydn.db.Storage('blog', blog_schema);
    blog_db.put('author', [author1, author2]);
    blog_db.put('article', article1, [author1.email, article1.title]);
    blog_db.put('article', article2, [author1.email, article2.title]);
    blog_db.put('article', article3, [author2.email, article3.title]);
    blog_db.put('article', article4, [author2.email, article4.title]);
           
Notice how out-of-line keys are constructed for 'article' store. This database library should automatically generated such parent-child relationship key.
 
Let us query how many 'javascript' articles.
            
    blog_db.count('article', 'tag', ydn.db.KeyRange.only('javascript')).done(function(count) {
      console.log(count)
    })
                
Here list article 'title' by Micheal Bolin in the database.
                
    blog_db.keys('article', ydn.db.KeyRange.starts(['bolinfest@gmail.com'])).done(function(keys) {
      for (var i = 0; i < keys.length; i++) {
        console.log(keys[i][1]);
      }
    });
                    
The following code snippet query primary keys sorted by index key 'title'.
                    
    blog_db.keys('article', 'title').done(function(keys) {
      for (var i = 0; i < keys.length; i++) {
        console.log(keys[i][1]);
      }
    });
                        
Many-to-many relationship
 
Many to many relationships are the most complex to implement in any system, and several solutions are available. The most obvious approach is the same as that used in relational databases: a 'join table', which contains pairs of keys from both sides of the relationship.  
                      
    friendship_store_schema = {
      name: 'friendship',
      indexes: [
        {
          keyPath: 'author1'
        }, {
          keyPath: 'author2'
      }]
    };
                          
Another approach is to have one side of the relationship store a list of the keys of records on the other side of the relationship. This makes the most sense when the cardinality on one side is limited (say, to a few hundred or less), when there is a natural 'owner' of the relationship, or when you want to be able to easily update the list of referenced entities in bulk.
 
This is illustrated by relationship between 'topic' and 'article' stores.    
                      
    topic_store_schema = {
      name: 'topic',
      keyPath: 'name'
    };
    article_store_schema = {
      name: 'article'
      indexes: [
        {
          keyPath: 'title'
        }, {
          keyPath: 'publish'
        }, {
          keyPath: 'license'
        }, {
          keyPath: 'publisher'
        }, {
          keyPath: 'topics',
          multiEntry: true
      }]
    };
                          
In the fugure, the library intent to provide relationships by annotating with the following keywords in the schema.                          
          
{% endwrap %} 