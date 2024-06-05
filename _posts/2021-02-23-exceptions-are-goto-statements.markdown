---
layout: post
title: "Abusing exceptions as goto statements"
date: 2021-02-25 09:00
comments: true
categories: [Java, Programming, Exceptions]
---

A reasonably common abuse I see is abusing exceptions as a form of control flow. If it's inside a single method, it's not too bad, but it's easy to get to the point where exceptions get thrown or re-thrown and is dealt with elsewhere in the code.

Let's look through an example.

```kt
interface PersonSerice {
    fun getPerson(id: Int): Person
}

class DbPersonService() : PersonService {
    fun getPerson(id: Int): Person {
        return db.fetchById(id)
    }
}


fun getPersonRequest(id: Int) {
    return try {
        200(service.getPerson(id))
    } catch (DbRecordNotFound: e) {
        404()
    }
}
```

It is not complex code; you can easily understand what's going on. But there's a lot of mixed responsibilities going on for such a small bit of code. 

The controller needs to know about the implementation details of the interface; expect it's worse than that; it needs to know about the implementation details of the library used in the service! In this case, it needs to know that it throws an exception of type `DbRecordNotFound` as opposed to say `FileNotFound`. 

If you look more closely at the `getPersonRequest` you will see that there's control flow logic hidden away in a simple catch statement. It's not too bad, but you can see how it could easily be worse.

### Wrapping Exceptions

To make things more consistent, developers will often catch and throw a more specific exception.

```kt
class FilePersonService() : PersonService {
    fun getPerson(id: Int): Person {
        try {
            return db.fetchById(id)
        } catch (FileNotFound: e) {
            throw PersonNotFound(e)
        }
    }
}

class DbPersonService() : PersonService {
    fun getPerson(id: Int): Person {
        try {
            return db.fetchById(id)
        } catch (DbRecordNotFound: e) {
            throw PersonNotFound(e)
        }
    }
}

fun getPersonRequest(id: Int) {
    return try {
        200(service.getPerson(id))
    } catch (PersonNotFound: e) {
        404()
    }
}
```

We haven't realy solved the problem. There's still a hidden goto statement when we throw the exception.

### Using nulls

So sometimes, developers use nulls to represent things that are not found. Personally, I think this isn't very good. But it's worth exploring. 

```kt
class DbPersonService() : PersonService {
    fun getPerson(id: Int): Person? {
        try {
            return db.fetchById(id)
        } catch (DbRecordNotFound: e) {
            return null
        }
    }
}

fun getPersonRequest(id: Int) {
    val person = service.getPerson(id)
    return if(person != null) {
        200(person)
    } else {
        404()
    }
}
```

We could use a ternary operator, or we could take advantage of some of the syntactic sugar in kotlin.

```kt
       
fun getPersonRequest(id: Int) {
    return service.getPerson(id)
        ?.let {
            200(it)
        } ?: {
            400()
        }
}

```
I think this horrible; we're just hiding the control flow again. If I'm honest, I prefered the null check.

### Default types
I think it's better to be explicit with control flow within your programs. I also think it's better to be explicit with what is returned from another class rather than abusing nulls.

```kt
class DbPersonService() : PersonService {
    fun getPerson(id: Int): Person {
        try {
            return db.fetchById(id)
        } catch (DbRecordNotFound: e) {
            return Person.NOT_FOUND
        }
    }
}

fun getPersonRequest(id: Int) {
    val person = service.getPerson(id)
    return if(person == Person.NOT_FOUND) {
        404()
    } else {
        200(person)
    }
}
```

I'm still not completely happy with this, but I think it's better than the alternatives.

## Conclusions

Stop abusing throw for control flow. If you're throwing up your code is sick!