---
layout: post
title: "How To: perform an 'in' query with mongo and spring-data"
date: 2020-03-04 08:00:00 +0000
comments: true
categories: ["developers", "spring", "spring-data"]
---

Unfortunately it looks like spring-data will not autowire up queries that will perform an `in` query under the hood. 

Out of the box, you can query for ids using a CrudRepository and `fun findAllById(ids: Iterable<ID>) : Iterable<Entity>`. Spring will automatically wire up `fun findAllByPropertyId(propertyId: String) : Iterable<Entity>` and do what you expect but what it won't do is wire up `fun findAllByPropertyId(propertyIds: Iterable<ID>) : Iterable<Entity>`. You must to annotate the interface method with the query you want it to perform.

```
@Query("{propertyId: { \$in: ?0 }}")
fun findAllByPropertyId(propertyIds: Iterable<ID>) : Iterable<Entity>
```