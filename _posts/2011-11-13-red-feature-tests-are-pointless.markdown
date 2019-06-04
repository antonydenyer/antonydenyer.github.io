---
layout: post
title: "Red feature tests are pointless"
date: 2011-11-13 17:52
comments: true
redirect_from: '/blog/2011/11/13/red-feature-tests-are-pointless/'
categories: [Feature Tests,Agile]
---

We've spent a lot of time recently on fixing up our [automated feature tests (AATs)][1]. The problem has been that these failing tests have blinded us to real problems that have crept into live systems. The usual answer is to 'just run them again' and eventually they go green. The main problem is our attitude to the tests, we don't respect them and we don't listen to them, as such they provide no feedback and are completely pointless. The response to broken feature tests would normally range from the test environment is down, the data is in contention, the database is down etc etc, but never something I've done has broken something.
So what is the solution? 

We improved the reliability of the tests that we could and removed the ones we couldn't. Now you may think this is a bit of a cop out, but the amount of feedback we were getting from the tests was negligible. The best think you can get from your tests is something you don't expect. you should expect them to pass and be surprised or shocked when they don't. 

Stop the line and fix the problem. Don't just keep running them again.

 [1]: http://www.extremeprogramming.org/rules/functionaltests.html  