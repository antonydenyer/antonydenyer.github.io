---
layout: post
title: "Feature Tests using Cucumber.js and zombie.js"
date: 2012-01-23 18:01
comments: true
redirect_from: '/blog/2012/01/23/featuretests-using-cucumber-dot-js-and-zombie-dot-js/'
categories: [Feature Tests,BDD,Node.js]
---

I wanted to start looking at alternatives to our current set of cucumber feature tests. At the moment on the web team we're using using FireWatir and Capybara. So I though I'd take at look at what was available in [Node.js][1]. Many people think it's strange that a .Net shop would use a something written for testing Ruby or even consider something that isn't from the .Net community.&nbsp;Personally&nbsp;I think it's a benefit to&nbsp;truly&nbsp;look at something form the outside in. &nbsp;Should it matter what you're using to drive your end product or what language your using to test it? Not really. So what are the motivations for moving away from Ruby, Capybara and FireWatir?
In a word 'flaky', we've had heaps of issues getting our feature tests, AATs and smoke tests reliable. When it comes to testing, consistency&nbsp;should be king. They should be as solid as your unit tests. &nbsp;If they fail you want to know that for definite you've broken something, rather than thinking it's a problem with the webdriver.

It is with this aim in mind that I started looking at the following.

[Cucumber.js][2] is&nbsp;definitely&nbsp;in it's&nbsp;infancy, there's lots of stuff missing but there's enough there to get going.

[Zombie.js][3] is a headless browser, it claims to be&nbsp;insanely&nbsp;fast, no complaints here.

First up we got something working with the current implementation of cucumber-js&nbsp;. The progress&nbsp;formatter&nbsp;works fine and the usual "you can implement step definitions for undefined steps" are a real help. Interestingly rather than requiring zombie.js in our step&nbsp;definitions&nbsp;we ended up going down the route of implementing our own DSL inside [world.js][4]. We could have used&nbsp;another&nbsp;DSL like capybara to protect us from changing the browser/driver we use. This is currently done with our Ruby implementation, the problem is that we've ending up implementing our own hacks to get round the limitations/flakiness&nbsp;of selnium/webdriver and to date we have never 'just swapped out the driver' to see what happens when they run against chrome/ie. That said should you be using cucumber tests to test the browser? I don't think you should. With that in mind we ended up&nbsp;implementing&nbsp;directly against [zombie.js][3] from our own DSL.

Extending&nbsp;cucmber-js 

There are a lot things yet to be implemented in cucmber.js one that gives me great satisfaction is the pretty formatter. Look everything is green!&nbsp; It's no where near ready for production but you do get a nice pretty formatter.

Thanks to [Raoul Millais][5] for helping out with command line parsing and general hand holding around JavaScript first steps.

 [1]: http://nodejs.org/
 [2]: https://github.com/cucumber/cucumber-js
 [3]: http://zombie.labnotes.org/
 [4]: https://github.com/antonydenyer/zombiejsplayground/blob/master/features/support/world.js
 [5]: https://twitter.com/#/raoulmillais  