---
layout: post
title: "Cache Fluffer / Gorilla Caching / Cache Warmer"
date: 2012-07-22 20:27
comments: true
redirect_from: "/blog/2012/07/22/cache-fluffer-slash-gorilla-caching-slash-cache-warmer/"
categories: [Cache]
---
The relatively simple introduction of a cache fluffer can make a huge difference to performance, particularly at peak load. The idea is simple, keep the cache up to do date so you don't have to go and get data when the user requests the site.

         public T Get(Func factory, string cacheKey)  
         {  
             if (cache.Contains(cacheKey))  
             {  
                 Task.Factory.StartNew(() = **FluffTheCache(factory, cacheKey)**);  
                 return cache.Get(cacheKey);  
             }  
             return Put(factory, cacheKey);  
         }
         public void FluffTheCache(Func factory, string cacheKey)  
         {  
             var expiry = cache.getExpiry(cacheKey);  
             if (expiry.Subtract(new TimeSpan(0, 0, 10)).Second &lt; new Random().Next(1, 60))  
             {  
                 Put(factory, cacheKey);  
             }  
         }
 
         public T Put(Func factory, string cacheKey)  
         {  
             var item = factory();  
             cache.Set(item, cacheKey);  
             return item;  
         } 

The beauty with this method is that popular items will always be in cache where as less popular items will not be cached for unnecessary lengths of time. 

You also get some randomization mixed in for free meaning that items don't come out of sync at the same time.  