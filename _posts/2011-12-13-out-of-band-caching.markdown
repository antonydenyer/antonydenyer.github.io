---
layout: post
title: "Out of band caching"
date: 2011-12-13 17:57
comments: true
redirect_from: '/blog/2011/12/13/out-of-band-caching/'
categories: caching
---
One of the things we try and do is always keep our site up, sounds simple right? One way to&nbsp;achieve&nbsp;this is to have a good caching policy.
Here's what we do:

*   Cache a response from some external service (be it a db or web service, whatever)
*   When a user request the resource ALWAYS serve it from cache if it's there
*   Then "out of band" (i.e. on a different thread) make a call to get it from the external service and update the cache if you get a valid response.
*   But only do this if the cache is stale Simple right? That way if your external service goes down you can still serve it from cache. Your user will always get the fastest possible response (i.e. they wont be the sucker who has to go and get it). And your service remains up as much as possible.  