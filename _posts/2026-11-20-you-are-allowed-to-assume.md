---
layout: post
title: "You Are Allowed to Assume"
date: 2026-11-20 09:00 +0000
comments: true
categories: [ai, programming]
---

AI hallucinations are just false assumptions.

Some people talk about it as though a model had found a new way to be wrong. A programmer who reads `CustomerUpdatedEvent` and takes it as a command has done the same thing.

Programmers have been making false assumptions for as long as someone else has had to read their code. The useful part of the craft is the work that stops the next reader doing it. That reader used to be a person. Now it is almost always an agent. A codebase has to be structured and organised so the reader can make the right assumption.

## When there was nothing to assume

Machine code gives you very little room for a false assumption, and very little room to understand anything you cannot hold in your head. Every instruction is on the page. You know exactly what the machine will do by reading all of it.

Then you hit a wall. You can't hold it anymore, so you make an abstraction. That's compaction. You skim what something does, and you assume the rest. An agent hits the same wall. We just call it a context window.

High-level languages were the first real abstractions. A function name stands in for a run of instructions. You call it, and most of the time you don't need to think too much. You're assuming the name is true. When you still have to open `sort` to find out whether it sorts, the name has failed, and you're back to reading every instruction with extra syntax in the way. We've known this for years, and we still struggle to teach it. `Sort` should do just that. It probably shouldn't be sending an email at the same time.

Objects and types took it further. Some state, and the behaviour you're allowed to use, behind an interface. You stop caring how the fields are laid out. You care what you're allowed to do with the thing.

Then the Gang of Four named a bunch of design patterns. Once you've seen a strategy, or a factory, you can assume the moving parts and spend your time and attention on the things that matter.

Craftsmanship, SOLID, DDD take that out to the shape of the system. A boundary is a statement about which question you've finished with. None of this is new. A good codebase has always been a machine for producing the right assumption, cheaply, and for making a wrong one look like a mistake in the code. We've had years of talks about code smells, best practices, naming conventions, DSLs and on and on. All so that we can collectively make assumptions.

## CreateCustomer

`CreateCustomer` will probably create and persist a customer. Which database? I don't care. How does it write it? I don't care. I'll care the day it goes wrong, and on that day I will have to go and look. Until then I can assume the outcome.

`CustomerUpdatedEvent` is news. Something happened, and this is the record of it. A handler that receives it is hearing about a change, after the change.

If `CreateCustomer` does not write a customer, the name is wrong, not the reader who believed it was doing what it said on the tin.

## Written down beside the code

Agents are trained on code written for humans. What `CreateCustomer` does will depend on the codebase! Sometimes it assembles a form and will return an instance of a `Customer` object. Sometimes it saves a `Customer` to the database. The name still invites the assumption. That's what names are for, but sometimes the assumptions we make require further understanding and context rather than just the name.

So the help has started turning up as a second literature. `AGENTS.md`. Rules files. ADRs whose job is to say which folder the handlers live in, and which layer is allowed to call which.

A good codebase was already supposed to do that with self-documenting code. When the names and the boundaries hold, a programmer can take the next step without a guide, and an agent can do the same. When they don't, a human can ask, but an agent can't or doesn't, so we codify it in markdown. We write it because this reader was trained on the repositories where the names don't hold, and because it still can't see the whole repository. It can't do what a good programmer does, which is go and check and ask.

## The context window

An agent's context window is often smaller than the codebase, so we need to leave maps and hints. A file at the root that says where things are. A compacted conversation, so the next turn still knows what the first turn decided.

I think the context window will grow, and soon models will be able to see the whole repository. Some of them nearly can, on a small enough codebase. A map of the tree has a short life once the reader can walk the tree.

A bigger window leaves `CustomerUpdatedEvent` meaning what the name says. A person could already open every file. Good programmers never grokked a system that way. They grokked it because the next assumption was obvious, and the code had been built so that assumption was the true one. A bigger window makes a lying name easier to catch, the same way a search box does. The name is still either doing its job or failing at it. How many "refactoring" PRs have you written where it's mostly name changes!?

## What is worth keeping

Long term, I think the maps will go, much like code comments. A file that says the handlers live in `src/handlers` is a second copy of something the tree already says, and it'll drift the week someone moves the folder. Once a model can read the repository, that file is a stale comment. Even if they "never forget" to update it, the comment is still mostly pointless.

The obligation stays, because it was never about the window. Any programmer or agent needs the obvious reading to be the right one. A `CreateCustomer` that does something else is a footgun waiting to go off.

An ADR earns the file when it records a decision the code can't carry. What we looked at, what we refused, and why. What we're not going to reopen, even though a new reader, human or otherwise, would quite reasonably have a go. That was the reason to write one down before anyone pointed a model at a repository. An ADR that only points at a directory was a map, not a decision.

## Do you still compact by hand?

It's been about twelve months since that was a normal thing to do. The files we're writing so a model can find its way around the codebase will age just as fast. But helping a reader make the right assumption won't. We did it for programmers, with names, boundaries, encapsulation, and abstractions.

Some of that help is in the code, where it always belonged. A lot of it, has moved from being implicitly agreed by teams to being explicit recorded stored in markdown sat next to the code.