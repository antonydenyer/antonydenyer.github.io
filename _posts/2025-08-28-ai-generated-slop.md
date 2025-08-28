---
layout: post
title: "AI Slop and Brandolini's Law"
date: 2025-08-28 09:00 +0000
comments: true
categories: [TDD, genAI, DDD, eXtreme Programming]
---

Brandolini's Law "the amount of energy needed to refute bullshit is an order of magnitude larger than to produce it" is about to hit our codebases with a vengeance.

As the use of AI code generators becomes more widespread, the amount of AI-generated code will increase. On the surface, the code will compile and superficially work, but will ultimately lack clarity and deeper architectural principles. The time and effort required to write a prompt and receive plausible code is measured in seconds. But the time it takes to review, understand, debug, and refactor that code to meet production standards is often measured in hours. This imbalance compounds quickly, with the majority of the work falling on senior developers to refute bullshit PRs. 

## Brooks Law
We've long known via [Brooks' Law](https://en.wikipedia.org/wiki/Brooks%27s_law) that adding people to a late project only makes it later. Imagine a three-person development team with two seniors and one junior. Now imagine adding three more juniors. Will it help? Probably not. The senior devs will drown in PR reviews, handholding, and rework.

This is precisely what's happening with AI-generated code. Only now, the "juniors" never sleep, and they can submit bullshit PRs at scale.

## What This Means for Testing 

Automated testing will become even more critical in a genAI world. Automated test suites will become the primary line of defence against code entropy and offer the only scalable way to refute bullshit code that fails to perform as expected. If you assume the volume of low-signal contributions will grow, the cost of not having strong tests will grow exponentially.

In this future, test coverage will not just be a quality metric; it will be the gatekeeping mechanism that enables you to move fast.

This shift changes the role of engineering managers and tech leads. It's no longer sufficient to encourage a "testing culture" or to insist on test coverage as a post-hoc check.

**Teams will need deliberate testing strategies designed to:**

- Detect subtle breakages in complex logic quickly.
- Prevent regressions from plausible-looking but semantically incorrect changes.
- Ensure long-term stability as the codebase absorbs AI-generated contributions.

## What does this mean for code quality?

I've always advocated for clean, maintainable code. Maintenance is where most of the real cost lies. But if code can be re-implemented in seconds, does that burden disappear?

Not exactly. The focus shifts.

Implementation details may become disposable. But **tests**, **interfaces**, and **system boundaries** become more important than ever — because they encode the actual intent.

## Extreme Programming and TDD Renaissance

It means designing systems with testability in mind from the start. You write the test by hand first, and then leverage GenAI to implement it. Programming becomes even more about understanding the problem you're solving and articulating that in a codified test. Then, structuring the communication between code boundaries. The isolated functional level implementation details will matter less. But the architectural and system boundaries will still be necessary.
Contamination of Training Data
When experimenting with code katas, I've noticed that models often take cues from the test names and over-implement, frequently exceeding what the test actually requires.

This mirrors a common mistake in TDD: coding ahead of the test, which kills emergent design.

In a similar vein, future AI models might learn to generate code that matches similar patterns from training data. This is test leakage akin to when test samples sneak into ML training sets, inflating performance metrics.

To counter this, we may need **test isolation strategies** to keep test suites private or out of reach of training pipelines. It's too early to say what best practices will emerge, but the risk is real.

## Conclusion

We're definitely headed for a future of *more* code. But, sadly, most of it will be poor-quality code. The craftsman in me weeps at the thought of AI-generated slop infecting my pristine codebases. Maybe I've just become the old-school engineer who scoffed at garbage collection, insisting real control meant managing memory by hand or "that guy" who rants about IDEs making people lazy, because "real programmers use Vim"?

Probably, but if "software is eating the world", it's genAI that has the biggest appetite and with the burden on humans to vet, test, and maintain that software. The teams without strong, intentional testing practices, will drown in bullshit.
