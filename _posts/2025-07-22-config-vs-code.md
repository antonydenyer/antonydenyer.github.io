---
layout: post
title: "Keep It in Code: Rethinking What We Externalise"
date: 2025-07-22 09:00 +0000
comments: true
categories: [agile, devops]
---

Most modern software projects are built with the assumption that configuration belongs *outside* of the code. If you've read [The Twelve-Factor App](https://12factor.net/config), you've likely seen the principle stated clearly: **store config in the environment**.

That guidance has stood the test of time - but like most well-intentioned principles, it often gets misapplied. Over time, "put all your settings in environment variables" becomes dogma, and teams end up with sprawling `.env` files, YAML configuration hell, and secrets and logic scattered across systems that no one fully understands.

So it’s worth asking: **what actually counts as configuration?** And more importantly: **what doesn’t?**

## The 12-Factor Definition

To start, here's the canonical definition from 12factor.net:

> "An app's config is everything that is likely to vary between deploys (staging, production, developer environments, etc). This includes:
>
> * Resource handles to the database, Memcached, and other backing services
> * Credentials to external services such as Amazon S3 or Twitter
> * Per-deploy values such as the canonical hostname for the deploy"

At a glance, this seems straightforward. Config includes values that vary across environments. However, it leaves some grey areas, which we'll explore. Especially around who manages what and how often those values change.

## A Practical Rule of Thumb: Who Owns the Change?

Let's consider a reframing of the question. Instead of asking "will this value ever change?", ask:

> **Who knows when this should change, and who is responsible for managing it?**

There's a meaningful distinction between:

- Values that **the dev team owns**, and understands deeply
- Values that **infra/ops/devops/SRE** teams manage, often across many services

Using this lens, a few things start to shift.

## What *Should* Be in Code

If a value is static, known at development time, and not expected to vary across environments **except as a result of deliberate code/product decisions**, it should live in code.

**Examples:**

- The base path of an internal API (e.g., `/v1/search`)
- The name of a queue or topic your code consumes (e.g., `user-notifications`)
- The URL of an internal service you own, assuming environments are discoverable (e.g., via service discovery or internal DNS)
- Feature flag keys (but not their *values*)

These are **development-domain values**. The dev team defines them, hopefully with the guidance of the wider business. The infra team might not even be aware of their existence. Putting them in configuration creates indirection without real benefit — and often, additional failure modes.

If a change to the value requires a corresponding change in code, it's not config. It's a **parameter** of the software, and should be treated like source code.

## What *Should* Be Config

Conversely, values that genuinely vary across environments or deploys — *without requiring a code change* — do belong in config.

**Examples:**

- Database connection strings and credentials
- External API tokens
- S3 bucket names (when per-env)
- Application-specific settings tuned by ops (e.g., cache TTLs, log levels, scaling thresholds)

The key here is **separation of responsibilities**. If DevOps/SRE is managing environments and needs to change something without involving the dev team, that's config.

## What About Rarely Changing Values?

Here's the murky part: some values don't vary per environment, but still **change occasionally** — maybe once every few months or years. Think:

- The canonical public hostname for your app
- The base URL of a third-party API that updates once in a blue moon
- The ID of a payment provider account

Technically, these could be in config. But in practice, putting them in config increases the number of places a developer has to look when debugging. It also weakens the guarantees that code works out-of-the-box, without knowing some invisible environment setting.

In these cases, a practical rule is:

> If the dev team needs to know about the change and needs to test it anyway, keep it in code.

This is where **version control** beats environment management. Code has history, context, review, and accountability. If you put a rarely changing value into a secret store or config map, you lose visibility into how and why it changed.

## The Cost of Over-Configuring

Here's what happens when config sprawl gets out of hand:

- Harder onboarding: New developers have to hunt down dozens of environment variables or config maps just to run a test.
- Fragile deploys: A missing or outdated variable in one environment can silently break things.
- Poor traceability: You see a misbehaving app in prod, but can't tell if the config changed or if the code is at fault.
- Broken assumptions: Tests pass in dev but fail in staging, because the "config" doesn't match.

Too often, the pendulum swings too far toward "make everything configurable" in the name of flexibility — but flexibility without clarity is a trap.

# Ownership Defines the Boundary

Here's a distilled principle I've found useful:

> **If the development team owns the value and it rarely (or never) changes per environment, it should be in code.**
>
> **If the infrastructure team owns the value and it may differ per environment or needs to change without a deploy, it should be in config.**

This leads to some conclusions that may run counter to so called "best practice":

- Internal service URLs? Put them in code.
- Feature flag keys? Code.
- Feature flag values? Config.
- Default cache TTLs? Config if ops needs to tweak them. Otherwise, code.
- Secrets? It depends on how secret it needs to be!

### Secrets: often aren't that secret

A common pushback I get on this is the issue of secrets. But really, how secret does that secret need to be? An API key for accessing read-only data? Code. Basically, if it's something you share with your team using 1Password that gets put into an env var, it's not really a secret, is it! 

## Closing Thoughts

The 12-Factor principle still holds water. But interpreting it blindly can lead to brittle systems and poor developer experience. Instead, draw the line based on *ownership* and *variance across environments*.

Keep things in code when they are known at build time and don't need to change per deployment.

Put things in config when they *must* be managed independently of the code — and ensure those changes are tracked.

More precise boundaries lead to more maintainable systems. And a happier dev team.
