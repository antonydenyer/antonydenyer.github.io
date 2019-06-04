---
layout: post
title: "Resolving an open generic type with Castle Windsor"
date: 2011-01-14 17:31
comments: true
redirect_from: '/blog/2011/01/14/resolving-an-open-generic-type-with-castle-windsor/'
categories: castle windsor
---

One of things I wanted to do the other day was resolve an open generic interface. Or more specifically resolve a generic type at runtime.
This is what we came up with:
```
    var argumentsAsAnonymousType = 
     typeof(IHandler)
      .MakeGenericType(instance.GetType());
    
    var concrete = 
      IoC.Container.Resolve(argumentsAsAnonymousType);
``` 
The first problem was resolving an open generic type, this was solved using [MakeGeneric][1]. This returns a type object which you can pass to windsor to resolve for you.
But now you're left in a situation where you don't know what the returning type is. You could use reflection to call the method. &nbsp;The good thing about this is in the type you're declaring you can afford to use a generic type. 
```
    public interface IHandler
    {
        void Handle(T instance);
    }
    
    public class FileNotFoundHandler : IHandler
    {
        public void Handle(FileNotFoundException exception)
        {
            _logger.Warn(fileNotFoundException.FileName)
        }
    }
```    
The downside is, you're using reflection! The alternative is to use a non generic interface and utilize interface inheritance. 
```
    public interface IHandler : IHandler
    {
    
    }
    
    public interface IHandler
    {
       void Handle(object instance);
    }
    
    public class FileNotFoundHandler : IHandler
    {
        public void Handle(object exception)
        {
            var fileNotFoundException as FileNotFoundException;
            _logger.Warn(fileNotFoundException.FileName)
        }
    }
``` 
That way when you resolve the handler you can cast it.
```
    var argumentsAsAnonymousType = 
     typeof(IHandler)
      .MakeGenericType(instance.GetType());
    
    var concrete = 
      IoC.Container.Resolve(argumentsAsAnonymousType) as IHandler;
    
    concrete.Handle(exception)
``` 
  
The benefit now is that you don't need to use reflection. The down side is that you need to unbox your object inside the handler, however this shouldn't be an issue as you've already guaranteed the type when you resolve the object.

 [1]: http://msdn.microsoft.com/en-us/library/system.type.makegenerictype.aspx  