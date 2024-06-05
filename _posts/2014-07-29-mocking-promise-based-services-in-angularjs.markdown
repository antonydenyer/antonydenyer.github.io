---
layout: post
title: "Mocking promise based services in AngularJS with Jasmine"
date: 2014-07-29 08:37
comments: true
redirect_from: '/blog/2014/07/29/mocking-promise-based-services-in-angularjs/'
categories: [AngularJS, Jasmine]
---

When I first got stated using AngularJS and I had a few troubles with mocking. Primarily with injecting a mock into a controller.

In this scenario we have a controller that uses a service that we've created. The service has some functions on it that return promises that have a result. 


{% gist 15da0436e69c88552d85 MyController.spec.js %}

The first thing I got stuck on was the Jasmine syntax. With dynamic languages many people feel that it's good practice to stub over the top of something that already exists. In a so called bottom up approach the service that we are about to use has already been implmented. As such when we come to use the service in our test all we want to do is mock the method that we're using. To do this in Jasmine we use the spyOn method. The caveat with this is that we're spying on something that already exists. 

Now in our scenario the service doesn't exists yet. So we need to createSpyObj instead. The interesting thing to note is the second parameter on the method that describes the functions that are available on the spy. 

The other trick I learnt was that we can use the q library from angular. We can get this injected into our test the same way as everything else. Now it's very simple to get the stub to return a promise.  


```
myService.getSomething.and.returnValue(q.when({ result: 'I Promise' }));

```

To inject the service into our controller we can now simply pass the spy object into the constructor. We then need to propagate promise resolution, to do this we can use the $apply() method on the scope. If you don't do this your promise will not resolve and therefore the 'then' method on your promise will not be called. 


{% gist dff322e01734bcafb47c MyController.js %}

The next thing I need to figure out is how to match method parameters. 