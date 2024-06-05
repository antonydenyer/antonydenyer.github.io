---
layout: post
title: "Null Object Patern And Kotlin"
date: 2019-08-12 15:12:09 +0000
comments: true
categories: ["kotlin", "null"]
---

By default, Kotlin is [null safe](https://kotlinlang.org/docs/reference/null-safety.html). It means that you have explicitly state that you want a type to be nullable. For example, the compiler will stop you from doing stupid things.

For instance you can't do `var name: String = null` you need to explicitly say you want your string to be nullable `var name: String? = null`. Then when you try and access it, the compiler will force you to act `Only safe (?.) or non-null asserted (!!.) calls are allowed on a nullable receiver of type String?`. 

## Example

Let's take a look at a more concrete example. Say we have some existing Java code that we want to interact with and can not change:

```kotlin
class Entity {
  var name: String? = null
}

class Service {
  private val entity: Entity? = null

  fun getById(id: String): Entity? {
    return entity
  }
}

```

The problem is that when we come to interact with the Entity, we still need to decide what to do. The null problem is getting leaked into our Kotlin code. Say we want to get the length of the name property. We would need to write something like `val length: Int? = service.getById()?.name?.length` but we're still leaking the nullable type even further into our code. We might do something a little nicer by using the elvis operator to specify a default `val length: Int = service.getById()?.name?.length ?: 0` so that we don't leak that null everywhere.

My personal preference would be to treat everything outside of Kotlin as an external integration point and wrap it. The problem is that you end up with some reasonably janky code. Not only do you need to make a shadow `Service` to be completely safe, you probably also need to shadow `Entity`.

```kotlin
class SafeService {
  private val service = Service()

  fun getById(id: String): SafeEntity {
    return service.getById(id)?.let { SafeEntity(it.name ?: "") } ?: SafeEntity("")
  }
}

data class SafeEntity(var name: String)

```

I don't know that it's any better. For me, the main benefit is having the default behaviour in a single place.
