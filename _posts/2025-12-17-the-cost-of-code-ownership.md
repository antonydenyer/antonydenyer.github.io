---
layout: post
title: "The Cost of Code Ownership"
date: 2025-12-17 09:00 +0000
comments: true
categories: [tdd, engineering, GenAI]
---

In [my previous post](/2025-12-16-tdd-in-the-age-of-genai.md), I argued that GenAI shifts the centre of gravity of testing. If large language models can quickly generate plausible implementations, then tests that merely restate the implementation in another form lose much of their value. The real question becomes: **what are we actually specifying, and at what level does that specification meaningfully constrain the system?**

This is where property-based testing becomes more interesting. I’ve traditionally associated it with functional programming and more formal, mathematically inclined systems and never used it on production projects. In the context of GenAI, though, it begins to look more like a practical response to a changing cost structure.

## The tension: behaviour and properties

Behavioural tests have always been a compromise. They’re easy to read and easy to write, but they encode behaviour indirectly through a small, often unrepresentative set of cases. In classic TDD, you typically start with the happy path and then add edge cases. When tests are written after the fact, they frequently end up asserting what the code already does rather than driving what the system *should* do.

GenAI amplifies this weakness. If an LLM can generate both the implementation *and* a handful of tests from the same prompt, those tests are no longer validating behaviour. They’re validating the implementation, a common mistake made by junior practitioners.

Property-based testing approaches the problem differently. Instead of enumerating examples, it asks:

> What must *always* be true, regardless of input?

Rather than checking individual paths, properties constrain the entire space of valid behaviour. They act less like regression checks and more like executable specifications.

## Properties as ownership of intent

The hardest part of property-based testing isn’t tooling or generators. It’s identifying good properties.

Good properties encode domain understanding. They capture invariants that remain true across refactors, rewrites, and changes in implementation strategy. Once codified, they become durable guardrails for junior developers, for your future self, and increasingly for LLM-generated code.

This is where the notion of ownership starts to matter. If GenAI dramatically lowers the cost of producing code, the value shifts to the constraints that shape it. Properties are one way of expressing intent at a level that survives cheap, abundant output.

I’m not convinced that property-based testing will replace other testing styles. But it’s easy to imagine a workflow where developers focus on defining the right properties, and GenAI expands those into large, high-coverage test suites. The asymmetry is the point: a small amount of carefully chosen input can generate a large amount of useful output.

## History rhymes

Back in the early 2000s, large amounts of software development were outsourced to offshore teams. The cost of output dropped, and teams suddenly had access to vast quantities of cheaply produced code, with mixed results.

What many projects learned, often painfully, was that the real work wasn’t in writing the code. It was in understanding user needs, defining constraints, and communicating intent clearly enough that others could implement it correctly.

The same dynamic is playing out again. As the cost of output approaches zero, the value of the input increases. The advantage will go to teams who can specify the right things, ask the right questions, and articulate the right invariants, not those who can produce more code.