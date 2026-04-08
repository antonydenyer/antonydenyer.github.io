---
layout: post
title: "ETH needs a supply cap at 128 million"
date: 2026-04-08 09:00 +0000
comments: true
categories: [ethereum]
---

### Summary

Ethereum’s monetary policy works well today, but it is still harder to explain than it needs to be.

We currently rely on:

- variable PoS issuance  
- fee burn via EIP-1559  
- and a dynamic equilibrium between the two  

This is elegant, but not simple.

**Proposal:** introduce a hard cap at **128,000,000 ETH**.

With the current supply at ~121M, this leaves ~7M ETH of headroom, while formalising the scarcity Ethereum is already converging toward.

---

## The rule

Add a single invariant:

> **If total supply ≥ 128,000,000 ETH, then issuance = 0**

- Below the cap → normal issuance rules  
- At or above the cap → no new ETH issued  
- Burn continues via EIP-1559  

This makes the system:

- bounded above  
- still responsive below the cap  
- strictly non-inflationary at the ceiling  

---

## Why this helps with ultrasound money

### 1. A simple Schelling point

- clean  
- easy to communicate  
- comparable to Bitcoin’s 21M  

Ethereum’s monetary policy is often “too clever,” which makes ETH harder to explain as a scarce asset. A hard cap removes that ambiguity.

---

### 2. From conditional to guaranteed scarcity

Today:

> ETH *can be* deflationary

With a cap:

> ETH supply is strictly bounded, and burn can push it lower

This turns ultrasound money from an emergent property into a **protocol guarantee**.

---

### 3. Alignment with reality

We are already:

- at ~121M supply  
- operating under low issuance  

The cap simply formalises an endpoint that is very unlikely to ever be reached.

So the question becomes:

> why not explicitly codify what is already true in practice?

---

## Conclusion

Ethereum already behaves like a scarce asset, but communicating this externally is difficult.

A hard cap makes this:

- explicit  
- enforced at the protocol level  
- trivial to explain  

> **Ultrasound money, with a hard ceiling.**


Cross post from 
https://ethresear.ch/t/eth-needs-a-supply-cap-at-128-million/24618