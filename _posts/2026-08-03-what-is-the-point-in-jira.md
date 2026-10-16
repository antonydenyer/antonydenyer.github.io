---
layout: post
title: "What is the point in Jira?"
date: 2026-08-03 09:00 +0000
comments: true
categories: [ai, tdd, programming]
---

For the record, **I HATE JIRA**. And most of the other task management tools that go with it. In my view, they tend to get in the way of collaborative working. The idea is reasonable enough, but the setup and the way people use it almost always nudge you towards a pit of failure.

At its best, a Jira ticket should facilitate a discussion. At its worst, it becomes a substitute for collaboration, an exercise in exhaustive documentation, or somewhere for ideas to go to die.

I’m yet to see an improvement on good old-fashioned index cards. You get a snapshot of the project without having to open a virtual board. There’s also an actual cost to adding something: getting off your backside, writing it down and deciding where to put it. A small pause to think before firing another ticket into the backlog of DOOOOM.

I can get on board with a task planner as a to-do list. A reminder to come back to something because you were in the middle of something else. You’re coding, you notice something that needs attention, and you leave yourself a note so you don’t lose track of it. Even then, a good developer will often just clean it up as they go. Creating a ticket can take longer than fixing the thing. Sometimes you’ve only done it to keep the scrum master happy.

The trouble starts when we confuse managing tickets with planning work.

I’ve seen plenty of planning sessions where creating the ticket apparently means we’ve done the thinking. We’ve written down a request, added some acceptance criteria, given it a priority and put it in a sprint. Job done.

Except quite a lot of the thinking used to happen after that.

You’d start coding and discover the requirement didn’t make sense. Or you’d misunderstood the problem. Or there was a much simpler way of solving it. Or, occasionally, that the thing you were building probably shouldn’t exist at all.

Coding was part of figuring out what we were actually doing.

Now it’s pretty easy to fire that ticket at an agent and ask it to implement it. The awkward bit is that handing over the ticket doesn’t make any of those questions disappear. It just means we can get to the wrong answer much faster.

Prioritisation has always been one of the hardest problems in our industry. When development is expensive, the bottleneck is obvious. You can’t build everything, so you have to choose. We’ve spent decades driving down the cost of implementation through better tooling, higher-level languages, open source, continuous integration, cloud infrastructure and shorter feedback cycles. AI pushes that cost down again. But if implementation gets really cheap, we’ve got a different problem. We can build far more crap than we ever could before.

That brings us back to what building software has always been about: solving people’s problems.

I’ve enjoyed the craft for what it is, and spent plenty of time sniffing my own farts about architecture. But the biggest buzz I get is still seeing something I’ve built from nothing in the hands of someone who finds it useful.

So how do you prioritise when the cost of implementation approaches zero?

You look at the other costs.

Understanding the problem takes time. So does working out whether it’s worth solving, and finding out whether the thing you’ve built has actually made anything better.

At its best, TDD has always helped with this. You’d write a test, build a bit, learn something and change your mind. The short feedback loop helped you work out what the software needed to do. AI can make implementation dramatically faster (do you even look at the code anymore?!), but we need to be careful we don’t throw the thinking away with it.

Take the most important problem and work through it. Spend time with the people who are dealing with that problem. If someone is willing to give you their time to explain what’s making their life difficult, that’s worth paying attention to. Understand the pain, try a solution and find out whether it helps before moving on to the next thing.

And be prepared to throw things away. If it’s cheap to build another version, there’s less excuse for hanging on to one that doesn’t help.

This is probably where my dislike of giant backlogs comes from. They give ideas a strange sense of permanence. Somebody thought something might be useful six months ago, therefore it sits there waiting for its turn. If nobody cares enough about it to talk about it anymore, bin it. Cheap implementation doesn’t make a giant backlog more sensible. If anything, it makes being ruthless about what we work on even more important. Otherwise we’ve just created an incredibly efficient machine for producing software nobody particularly wanted.

So what’s the point in Jira?

A reminder, a prompt for a conversation, somewhere to record what we’ve decided. Fair enough.

But a ticket isn’t proof that something is worth building. And moving it to Done isn’t proof that we’ve solved anything.

As implementation gets cheaper, we need to be more rigorous about what gets that far in the first place. Be aggressive about deciding which problems are worth exploring, and just as willing to bin them when they don’t stand up.

A prioritised backlog of things nobody needs is still a backlog of things nobody needs.

Oh, and AI-generated tickets? Now we can fill the backlog of DOOOOM without even having to type.
