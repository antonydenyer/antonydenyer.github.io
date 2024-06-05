---
layout: post
title: "ElasticSearch schema migrations with zero downtime"
date: 2016-12-04 21:49:07 +0000
comments: true
redirect_from: '/blog/2016/12/04/elasticsearch-schema-migrations-with-zero-downtime/'
categories: ["elasticsearch", "schema mirgation", "re-indexing", "re-ingestion"]
---

One of the frustrations of working with elasticsearch is not being able to make changes to your mappings and indexes without being destructive. You'd never use elasticsearch as your source of truth, so this isn't normally a huge problem. But it can become a bit of a headache. The general advice tends to be 'make your changes and then re-index everything again.' Done wrong this can mean your search has no results while you re-index your data.

### How do you re-index data? ###
There are two ways of re-indexing your data.

* re-index data from your source of truth i.e re-ingest everything
* re-index data into a new index from the old index

What we're going to explore is how we re-index data into a new index from the old. There are some recommended practices in [https://www.elastic.co/guide/en/elasticsearch/guide/current/index-aliases.html](Elasticsearch: The Definitive Guide) that we're going to be expanding upon.

#### Architecture ####
The sort of system we will be talking about is one that follows [Command Query Separation](https://en.wikipedia.org/wiki/Command%E2%80%93query_separation). Essentially you have a system that reads from the elasticsearch and a separate system that writes to elasticsearch. Something like a back end catalogue editor and a front end e-commerce site.

### Use Aliases ###
Lets say you have an index called ```catalogue-v1``` you'll need to create two alliases that point to this index, ```catalogue-read``` and ```catalogue-write```. Change your application so that your front end application queies from ```catalogue-read``` and your back office points to ```catalogue-write```. 

When you have a breaking change to your schema you can create a new index ```catalogue-v2``` then modify the ```catalogue-write``` alias to point to the new index and the old one.

### Migration ###
If you've changed a data type or an analyzer you can tell elasticsearch to [re-index](https://www.elastic.co/guide/en/elasticsearch/guide/current/reindex.html) everything using the [re-index api](https://www.elastic.co/guide/en/elasticsearch/guide/current/reindex.html). Underneath the hood, it uses scroll api and the bulk api to push the data into a new index. If you need to change the data as you re-index you should use the scroll api and the bulk api. When you're happy with the new index you can modify the ```catalogue-read``` alias to point to ```catalogue-v2```.  

Then all you have to do is clean up the aliases and delete the old index.
