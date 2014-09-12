---
layout: ydndb-article
description: "Full text search module for YDN-DB"
introduction: "Explain mechanics of full-text search and introduce terms and concepts."
title: Basic concept
article:
  written_on: 2012-04-01
  updated_on: 2014-04-28
  order: 1
collection: ydndb-full-text
authors:
  - kyawtun

---

{% wrap content %}

## Mechanics of full-text search

YDN-DB uses an inverted index structure to store full-text index data. The inverted index structure is built by breaking searchable content into word-length _tokens_ (a process known as _tokenizing_) and storing each word with relevant metadata in the index. An inverted index for a document containing the phrase "My name is Peter." would be tokenized into four words: 'My', 'name', 'is' and 'Peter'. 

YDN-DB full-text search module use [Rodrigo Reyes](https://github.com/reyesr)'s excellent [fullproof](https://github.com/reyesr/fullproof) library for tokenization. It is a unicode-base tokenization supporting full language spectrum. 

The key fields in the inverted index include the word being indexed, a reference back to the source document where the word is found, and an occurrence indicator, which gives a relative position for each word. Full-text search indexing eliminates commonly used stopwords such as the, and, and of from the index, making it substantially smaller. With pre-defined stopwords removed, the inverted index for the previously given sample phrase becomes only two tokens 'name' and 'Peter'. Currently YDN-DB only have stop words for English language, which is based on [University of Neuchatel](http://www2.unine.ch/)'s [J. Savoy's compilation](http://members.unine.ch/jacques.savoy/clef/).

Whenever you perform a full-text search in YDN-DB, the full-text query engine tokenizes your input string and consults the inverted index to locate relevant documents. 

### Indexing process

When records of an full-text search object store is created or modified, indexing process initiated after storing the records. The database request is return only when indexing is finished.

The indexing process consults stoplists to eliminate stopwords from the tokenized content, normalizes the words, and adds the indexable words to inverted index fragments. The indexing process, in general, is CPU and I/O intensive. Despite the intensity of the process, the indexing process doesn’t block queries from occurring. 

{% include modules/remember.liquid title="Tip" inline=true text="Use separate transaction thread for populating full-text search object store, so that indexing is not interfered with UI request." %}
 
### Query process
 
The full-text query process uses the same word breakers that the indexer uses in the indexing process; however, it uses several additional components to fulfill query requests. The query processor accepts a full-text query predicate, which it tokenizes using word breakers. During the tokenization process, the query processor creates generational forms, or alternate forms of words using _stemmers_.
 
For a given word, stemmers return language-based alternative word forms, to generate inflectional word forms. These inflectional word forms include verb conjugations and plural noun forms for search terms that require them. Stemmers help to maximize precision and recall, which we’ll discuss later in this chapter. For instance, the English verb eat is stemmed to return the verb forms eating, eaten, ate, and eats in addition to the root form eat.
 
YDN-DB use stemmers from [Natural library](https://github.com/NaturalNode/natural). It supports several language, but currently YDN-DB only has English portion.

After creating generational forms of words, the query processor provides key for inverted index to retrieve matched records. The full-text query processor consults the full-text index to locate docu- ments that qualify based on the search criteria, ranks the results, and return back the relevant result to the user.

The full-text query processor is tightly integrated with YDN-DB core using private API and database request hook.

### Search quality

The quality of search results can be measured using two primary metrics: _precision_ and _recall_. Precision is the number of hits returned that are relevant versus the number of hits that are irrelevant. Precision can be defined mathematically using the formula as p = n / d, where p represents the precision, n is the number of relevant retrieved documents, and d is the total number of retrieved documents.

Recall is the number of hits that are returned that are relevant versus the number of relevant documents that aren’t returned. That is, it’s a measure of how much relevant information your searches are missing. Precision and recall are normally used in tandem to measure search quality. 

YDN-DB uses [Dice co-efficient](http://en.wikipedia.org/wiki/S%C3%B8rensen%E2%80%93Dice_coefficient) for measuring word similarity. [Term frequency–inverse document frequency](http://en.wikipedia.org/wiki/Tf%E2%80%93idf) is used to measure how important a word is to the document collection. Finally result is scored with positional weighted measure.  

{% endwrap %}   