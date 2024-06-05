---
layout: post
title: "Double Entry Accounting and TDD"
date: 2011-11-24 17:54
comments: true
redirect_from: '/blog/2011/11/24/double-entry-accounting-and-tdd/'
categories: TDD
---

[Double-entry bookkeeping system][1]  
A double-entry bookkeeping system is a set of rules for recording financial information in a financial accounting system in which every transaction or event changes at least two different nominal ledger accounts.  
At it's simplest you have two ledgers when you make an account transaction you make an entry in both ledgers. Then at the end of the month you reconcile these two ledgers and they should be the same. Essentially from an accounting point of view we're saying that by using two different ways of doing something we come to the same answer.
Why is the useful from a TDD perspective? 

Well you should be using different ways to make your assertions from your implementation, particularly when you talk about integration tests.

Lets say your writing some integration tests for a repository. Lets say you're using [Simple.Data][2] to access your database. Your tests should then use something else. You should probably get as close to the metal possible. In this case you should probably use the SqlCommand class. 

Put some data in your database using SqlCommand. Then assert that you can retrieve it using [Simple.Data][2]. 

That way you are making a **Double Entry Assertion** in your tests. The chances of both systems being broken in the same way are reduced.

 [1]: http://en.wikipedia.org/wiki/Double-entry_bookkeeping_system
 [2]: https://github.com/markrendle/Simple.Data  