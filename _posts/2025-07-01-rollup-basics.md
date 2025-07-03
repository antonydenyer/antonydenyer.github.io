---
layout: post
title: "Rollup Basics: Why Layer 2 Depends on Layer 1"
date: 2025-07-01 09:00 +0000
comments: true
categories: [ethereum, rollups]
---


Over the past few years, Ethereum has adopted **rollups** as its primary path to scalability. It's easy to get lost in all the jargon from zk-rollups, optimistic rollups, blobs, sequencers, based rollups, preconfs etc etc. A straightforward idea holds true.

> A rollup’s **real state** only exists when its data lands on **Ethereum Mainnet aka L1**.

In this post, I'll explain how rollups work and why the **safe state** of the chain in systems like OP Stack **only progresses when Ethereum includes the transaction batch**.

---

## What Is a Rollup, Really?

At its core, a **rollup** is a Layer 2 blockchain that *outsources its security* and *data availability* to L1. It runs its own virtual machine (often EVM-compatible), lets users transact cheaply and quickly, but regularly posts some form of **compressed transaction data** to Ethereum.

Rollups are faster and cheaper because they don't run every transaction on Ethereum itself; they simply *prove* or *attest* that they did, using on-chain data.

---

## Rollups Post Data to Ethereum

To be secure, a rollup must **anchor itself to Ethereum** by regularly posting:

- A **batch of transactions** (as calldata or in a blob)
- And often a **state root**, summarizing the updated L2 world state

Whether you're using an optimistic rollup (like OP Stack or Arbitrum) or a validity rollup (like Linea or zkSync), this L1 posted data is the *source of truth*.

Even if the sequencer *shows* your transaction in an L2 block, it's not truly canonical until Ethereum includes it.

---

## Safe Head vs. Unsafe Head

In the OP Stack, the rollup client defines three key "heads":

| Head Type       | Description |
|-----------------|-------------|
| **Unsafe Head** | Latest block built by the sequencer. Fast, but not yet backed by L1. |
| **Safe Head** | Latest L2 block derived from a batch that has **been posted and confirmed on Ethereum**. This is the **canonical L2 state**. |
| **Finalized Head** | A safe head rooted in a **finalized** Ethereum block (~64–128 sec). |

The **safe head** is what you should trust; it's derived entirely from L1 data. The unsafe head is speculative or optimistic!

---

## You Need to Wait for Ethereum to Include the Batch

If you're only seeing the **unsafe head**, you're in the world of "eventual correctness." It's real only if it later becomes part of the **safe head**.

### Example: How OP Stack Derives the Safe Head

1. The sequencer accepts user transactions and builds a batch.
2. The batch is posted to Ethereum (as a **blob**).
3. OP Stack nodes monitor Ethereum, download the blob, and **replay the transactions**.
4. After verifying the data, the client advances its **safe head** to reflect the new canonical state.

Until that blob is included in an Ethereum block, **no safe L2 state transition has occurred**.

---

## What This Means for Builders

If you're building on OP Stack or any modern rollup, keep this in mind:

- **Don't trust the unsafe head** for anything security-critical (like bridging funds or submitting fraud proofs).
- **Watch the safe head** for canonical state. That's the part of the L2 chain that *actually happened*, provably, on Ethereum.
- Build with an understanding that **latency is not instant finality**. There's speed from the sequencer, but there's trust from L1.

## Understanding the Optimistic Assumptions

It’s also worth understanding the optimistic security assumptions underpinning OP Stack. Optimistic rollups assume batches posted to Ethereum are valid, but allow anyone to challenge them if they suspect fraud. This is handled through the **fraud-proof game**.

### How the Fraud Proof Game Works

1. **Batch Submission:** The sequencer posts a batch of transactions to Ethereum.
2. **Challenge Window:** After posting, a fixed period (usually 7 days) is established, known as the **challenge window**. During this time, anyone can submit proof to show the batch is invalid and make a challenge.
3. **Fraud Proof Game:** If challenged, an interactive game is played on Ethereum to pinpoint the exact step where the state transition is incorrect. If fraud is proven, the invalid batch is reverted, and the sequencer may be penalised.

**In summary:**  
You can't trust anything on optimism until 7 days have passed!

## Validity Rollups

Validity rollups do not have the same concept of "safe" vs "unsafe" heads as optimistic rollups. Each batch posted to Ethereum includes a cryptographic proof that immediately proves the correctness of the state transition. Once the proof is verified on Ethereum, the new state is immediately final and canonical. There's no period where the state is "unsafe" or pending further confirmation. There's no need to wait for a challenge window or for L1 to "confirm" the batch beyond proof verification. Proof verification is reasonably fast (almost real-time); it's the proof generation that takes time. While things are getting faster with teams like RISC Zero and Succinct pushing things forward, we still don't have real-time proof generation.   

While validity rollups don’t require a challenge period, finality still depends on the L1 verifying the proof. Until then, the new state isn't canonical. This means that the same holds true as it does for optimistic rollups: you need to wait for finalisation before you know something has definitely happened, i.e., you need to wait for withdrawals.

This reliance on L1 data for proper finality introduces some latency and complexity, but it's essential for security and trustlessness. However, it also highlights a key challenge: how can we make L2s feel as seamless and secure as using Ethereum itself, without sacrificing these guarantees? 

Based rollups tightly couple block production with Ethereum’s own proposer, eliminating the sequencer trust assumption. This is precisely why **based rollups** can unlock so many opportunities by being based on the same block as mainnet.

## Based Rollups Can Make L2s Feel Like Ethereum Mainnet

A design where an **L2s state is derived solely from L1 confirmed data** is what makes a based rollup inherit Ethereum's security guarantees.

It might add a bit of ux latency to the L2, but it gives you something no sidechain or database-style L2 can offer: **trustless correctness**. 

It's this trustlessness that can unlock other, more interesting scenarios. Currently, to do anything on an L2, you need to trust the L2 (to varying degrees) while you are interacting on that L2. By removing that trust and delay, you can build a highly composable financial infrastructure that eliminates the need for intermediaries. Once you've done that cross chain syncronous composability is a hop skip and jump away.

With more blobspace on the horizon, rollups are increasingly able to anchor securely without breaking the bank - paving the way for higher L2 throughput and lower latency between unsafe and safe heads.

The ticker is $ETH
