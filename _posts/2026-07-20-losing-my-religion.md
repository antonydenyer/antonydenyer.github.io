---
layout: post
title: "Losing My Religion"
date: 2026-07-20 09:00 +0000
comments: true
categories: [ai, tdd, programming]
---

One of the things I’ve struggled with recently is getting back into a [state of flow](https://www.goodreads.com/book/show/66354.Flow).

For a while I assumed it was simply because I write less code than I used to. Like many engineering managers, my days are now split between product, people, planning, firefighting, and the occasional chance to write some real code. Those uninterrupted afternoons of building things have disappeared.

I think AI has fundamentally changed what it feels like to program.

This isn’t a criticism of AI. I’m not usually one for hyperbole, but I think it may be one of the biggest shifts our industry has ever seen. It allows us to build products faster, explore ideas more cheaply, and put useful features in front of customers in a fraction of the time. That’s [the goal](https://www.goodreads.com/book/show/113934.The_Goal?) right? Remove bottlenecks, add value.

When I do some programming, I’m more productive, and yet somehow programming feels slower.

## Reducing friction, increasing velocity

Looking back over the last twenty-five years, the story of software engineering—and my career—has largely been about removing friction.

When I first started, a large part of the challenge was simply figuring things out. Programming felt mathematical. You explored a problem, experimented with different approaches, and eventually discovered something that worked. As the industry matured, the challenge changed.

Initially, the web meant you could read the docs (MSDN was great then) without flicking through a book. Later, we got things like CodeGuru and Stack Overflow, which meant we no longer needed to memorise everything. Open source meant we could build on other people’s work instead of reinventing it.

Continuous integration reduced the time between writing code and knowing whether it worked. Continuous deployment reduced the time between finishing a feature and getting it into customers’ hands. Test-driven development encouraged tighter, more local feedback loops. In parallel, frameworks like Ruby on Rails made building web applications dramatically quicker. Stripe removed months of work from accepting payments. Every step made iteration faster. The feedback loop kept shrinking.

Even after programming shifted more towards plumbing, it still felt like momentum. When you’re deeply engaged with a codebase, you’re constantly building a mental model. You explore. You follow a few functions. You notice a pattern. You realise a piece of code wants reshaping. You refactor it. You make another small change. Everything feeds the next thought.

There’s a rhythm to that process. Small actions produce immediate feedback, which leads to the next action. Before long you stop thinking about typing altogether and you’re simply solving the problem. I miss that. Call it craft, call it artistry, call it the pleasure of making something real.

## AI changed the feedback loop

What’s different with AI isn’t that it writes code. It’s that it inserts a bottleneck into a process we’ve spent decades trying to remove.

We now have:

Think → Prompt → Wait → Review → Prompt → Wait → Review

Those pauses break your concentration. While the model is generating, you glance at Slack, at Twitter, or maybe you try to multitask. By the time the response arrives, you’ve lost the mental model you were building. You’ve destroyed your own context while the AI was building its own.

I think we’ve accidentally started using AI in a way that replaces the most enjoyable part of programming. Too often we treat it like an outsourced implementation team. We write a poorly defined spec, throw it over the wall to a cheap offshore “cloud team”, they come back with a solution, we review it, and then we repeat. **It’s not programming. It’s project management.**

It’s unsurprising that this feels hollow, because we’ve taken ourselves out of the conversation with the code.

I think AI should feel more like pairing than delegating. Good pair programming isn’t one person disappearing for half an hour and returning with a pull request. You’re both looking at the same code. The navigator asks questions while things are being implemented. You challenge assumptions, explore alternatives, and make small changes together. Most importantly, neither person leaves the problem, and both maintain context.

That’s how I think it should feel. We’re not there yet.

That said, I’ve had good experiences when I ask it to explain an unfamiliar part of the codebase, or explore a design together and discuss trade-offs. What I haven’t seen is a genuinely pair-programming-like AI experience yet.

