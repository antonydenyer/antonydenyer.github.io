---
layout: post
title: "TDD in the Age of GenAI: Behaviour Moves, It Doesn’t Disappear"
date: 2025-12-16 09:00 +0000
comments: true
categories: [tdd, engineering]
---

There’s a growing assumption in software engineering circles that GenAI will make traditional testing practices, and perhaps development, less relevant. Test-Driven Development has been on the chopping block for a while now. If a model can generate working code, scaffolding, and tests on demand, why insist on writing tests first? Why spend time encoding intent so explicitly?

I think this framing is fundamentally wrong.

What’s actually happening is not the erosion of TDD, but a **shift in where behavioural intent is expressed**. The tests don’t disappear; they move. And the role of engineers evolves from writing low-level assertions to deliberately designing, constraining, and validating system behaviour at higher levels.

## The Real Value of TDD Was Never the Tests

TDD has always been easy to caricature as “slowing software development down and writing lots of tests”. In my experience, its value comes from something more subtle and more important: **forcing behavioural clarity before implementation**. 

The tests are just the artefact. The thinking is the point.

This matters because GenAI models are exceptionally good at producing *plausible implementations*, but they are largely indifferent to *correct intent* unless that intent is clearly and explicitly expressed by the prompt. 

In essence, your tests have shifted to ephemeral prompts.

## Where GenAI Changes the Equation

GenAI models are already very good at:
- Generating implementation code from natural language descriptions
- Producing unit tests that mirror the implementation
- Filling in boilerplate test cases with high coverage but little insight

This is where many people conclude that TDD is dead. If the model writes both the code and the tests, the feedback loop collapses into self-confirmation.

But that critique only applies if your tests are **structural** rather than **behavioural**.

In practice, most AI-generated tests today assert *how* the code works, not *what the system guarantees*. They test methods, not meaning. They encode implementation details, not intent.

## Behavioural Tests Become More Important, Not Less

As GenAI takes over more of the mechanical implementation work, the tests that matter shift upwards:
- API-level behaviour
- Domain invariants
- Cross-service interactions
- Failure modes and edge cases
- User-visible outcomes

These are precisely the areas where humans still have a strong advantage, because they require context, judgement, and an understanding of real-world consequences.

GenAI is excellent at filling in behaviour once it’s specified. It is much worse at deciding *which* behaviour is correct.

That makes behavioural e2e tests the primary line of defence in an AI-accelerated codebase.

## Designing for This, Deliberately

What we’re only just beginning to learn is what this looks like when designed *intentionally* and *generally*, rather than as an afterthought.

There’s a design space here that goes well beyond “AI writes code” or “AI writes tests”, and it’s surprisingly underexplored. Almost no one is doing this today in the environments where it matters most: large, expensive, long-lived systems.


## A Likely Future: TDD as Behavioural Specification

I expect a common workflow to emerge that looks something like this:

1. Engineers define high-level behavioural tests or executable specifications.
2. These tests describe system boundaries, invariants, and failure modes.
3. GenAI generates large portions of the implementation to satisfy them.
4. Humans review behaviour, not syntax.

Seen this way, GenAI doesn’t replace TDD; it **amplifies its original purpose** by making it cheaper to enforce behavioural constraints consistently.

## The Real Risk: Scaling the Wrong Thing

The dangerous failure mode here isn’t “AI replacing TDD”. It’s **AI reinforcing shallow testing practices at scale**.

If teams accept:
- Auto-generated tests without behavioural intent
- Coverage metrics as a proxy for correctness
- Tests that restate implementation details

Then TDD becomes theatre.

But that was already true before AI. GenAI just makes it easier to do the wrong thing faster.

## Long-Term Maintainability Still Depends on Intent

The systems that will survive AI-accelerated development are not the ones with the most generated code. They’re the ones where:
- Behaviour is explicitly constrained
- Assumptions are executable
- Failure is intentional, not accidental

TDD done properly has always been about this. The presence of GenAI doesn’t remove the need. It changes *where you apply the pressure*.

The future isn’t test-free development. It’s **intent-first development**, with AI doing the typing.