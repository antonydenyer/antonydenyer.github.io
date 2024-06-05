---
layout: post
title: "Mongo $or queries are slow"
date: 2020-03-17 08:00:00 +0000
comments: true
categories: ["mongo", "data modeling"]
---

Mongo doesn't appear to perform particularly well when querying large-ish collections with $or queries. For some reason, it can't figure out which index to use and often ends up doing a COLLSCAN. Say you have the following document structure representing user transactions:

```
transactions collection:

{
    _id: ObjectId,
    toUserId: String,
    fromUserId: String,
    transactionData: String
}
```

Now say want to find all the transactions that a userId has performed. It's fairly simple to create a query for this:

```

db.transactions.find({$or:[{toUserId:"id1"}, {fromUserId: "id1"}]})

```

Queries like this are fine when dealing with relatively small result sets, but when it gets bigger, you end up with timeouts. With our setup we found that things stopped working when the result set got into the thousands of documents.  

The solution we came up with was to create our own index collection that we could perform the initial filtering on, then perform a second query to return the result set. The problem with this solution is that you are de-normalizing the data yourself and are performing two writes for each user transaction.

Here's what our index looks like:

```
transactions-index collection:

{
    _id: ObjectId,
    transactionId: ObjectId,
    userId: String
}
```

Now we can perform something like the following pseudocode:

```
val results = db.transactions-index.find({userId: "id1"})

val ids = results.map { x => {_id:x.transactionId} }

db.transactions.find({$or:[ids]})


```

The reason this works is that you are limiting the number of results that you want to look at in the initial find. Meaning the second `$or` query will only ever return a maximum of the page size. It also means you're always going to hit the IXSCAN on the transaction find. It also makes it easier for mongo to figure out which index to use for the initial transactions-index find.

The only downside is that you will end up returning the transaction twice if someone sends something to themselves! 