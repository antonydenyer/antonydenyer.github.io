---
layout: post
title: "Stub responses with Nest.ElasticClient"
date: 2012-08-01 12:08
comments: true
redirect_from: '/blog/2012/08/01/stub-responses-with-Nest.ElasticClient/'
categories: [Elastic Search]
---

We've started using [ElasticSearch][1] at work for some of our projects. When we started out doing simple web requests was easy enough but as the complexity of what we where doing grew it became obvious that we where starting to write our own DSL for elastic search. Surely someone else has already done this?

There are several options but we settled on [NEST][2]. Mainly because of the way the DSL was written. The first stumbling block I hit was unit testing. You could just stub out [IElasticClient][3] and then add some integration tests to cover yourself.

The problem with integration tests is they're slow and require, well something to integrate with, so I thought it would be good to stub the json response from ElasticSearch.

When using [NEST][2] it's not all that obvious. But here's how I did it:

{% gist 3225770 %}


The down side is you're testing the frameworks assumptions. But it's good to know what the framework is actually doing under the hood i.e. calling a web request with some parameters.

You also get the added bonus of being able to test that the response can be parsed into you're objects.

Doing to much in one test?

Absolutely, in our code base it's two tests. One for parsing the response and another test to assert post parameters.

  [1]: http://www.elasticsearch.org/
  [2]: https://github.com/Mpdreamz/NEST
  [3]: https://github.com/Mpdreamz/NEST/blob/master/src/Nest/IElasticClient.cs