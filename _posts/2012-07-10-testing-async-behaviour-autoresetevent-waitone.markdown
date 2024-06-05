---
layout: post
title: "Testing Async Behaviour - AutoResetEvent WaitOne"
date: 2012-07-10 20:23
comments: true
redirect_from: '/blog/2012/07/10/testing-async-behaviour-autoresetevent-waitone/'
categories: [async,TDD]
---

This little nugget helps you test async code. 


Essentially you use a semaphore to signal that something has finished. In our case we're using the [AutoResetEvent][1]. This allows to wait up to an amount of time for something happen.

In our case we're waiting for an action to be fired and we want to ensure the result is true.

     [Test]
     public void AsyncTest()
     {
       var autoResetEvent = new AutoResetEvent(false);
       new Something.Async(
         action =>
         {
            Assert.That(action.SomethingToTest, Is.True);
            autoResetEvent.Set();
         });
     
       var result = autoResetEvent.WaitOne(1000 * 60);
       Assert.That(result, Is.True, "Method Not Fired");
     } 

Make sure that you test the result from WaitOne. If you don't you're not testing that the method actually returns.

 [1]: http://msdn.microsoft.com/en-us/library/system.threading.autoresetevent.aspx  