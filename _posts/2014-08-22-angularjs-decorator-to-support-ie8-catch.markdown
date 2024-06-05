---
layout: post
title: "angularjs decorator to support ie8 catch"
date: 2014-08-22 16:09:29 +0100
comments: true
redirect_from: '/blog/2014/08/22/angularjs-decorator-to-support-ie8-catch/'
categories: angularjs
---
When using the angularjs $q library with ES3 browsers (IE8 etc) you get an 'Expected identifier' error when you try and use the catch method on angularjs $q library. This is because catch is a reserved word in ES3 and reserved words are not supported as property names in ES3. To get round the problem you can access the function using square bracket notation. Personally I find this kind of syntax rather ugly. 

```
	doSomething()
	.then(doSomethingElse)
	["catch"](allTheThings);
```

The first thing I wanted to do was provide a nicer syntax. The [following artice](http://dorp.io/blog/extending-q-promises.html) has a good explanation of how to create a decorator  for the $q service so that you can add your own methods onto it. The problem with the $q service is that you need to decorate the thing that gets passed back from the functions 'defer', 'when' 'reject' and 'all'. Rather than just putting it on $q. 

{% gist bac3b5245713d670d6e9 qDecorator.js %}

This means that we can now call

```
	doSomething()
	.then(doSomethingElse)
	.error(allTheThings);
```

and IE8 wont complain about it. Wonderful. The other thing I wanted to do was to make sure that everyone uses error instead of catch. So to guide people into this I've re-implemented catch to throw an exception. 