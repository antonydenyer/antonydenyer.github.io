---
layout: post
title: "Censorship Resistance Through Game Theory"
subtitle: "An Inclusion Lists Dilemma"
date: 2024-03-22 09:00
comments: true
categories: [block builder, mev, ethereum, PBS]
---

Censorship resistance is a fundamental property that underpins the decentralised nature of blockchain networks and ensures the integrity and accessibility of transactions within the system. While the threat of "hard censorship" is well understood, the challenges faced by preventing "soft censorship" resistance are more nuanced.

"Soft censorship" involves delaying or slowing down the propagation and inclusion of valid transactions. It could be accidental or deliberate; either way, it disrupts the normal flow of transactions, negatively impacts execution quality and harms user experience.

"Hard censorship" means the complete and permanent prevention or blocking of valid transactions from landing on-chain (e.g., OFAC compliance). The majority of research on censorship resistance has focused on this topic.

Previous approaches to mitigating censorship have focused on inclusion lists and tend towards some form of unconditionality. A proposer must include a transaction, or it is a protocol violation. 

[https://ethresear.ch/t/unconditional-inclusion-lists/18500](https://ethresear.ch/t/unconditional-inclusion-lists/18500)

By applying game theory principles to PBS, incentives can be leveraged to foster etherum aligned outcomes. Our goal is to create a scenario where including all valid transactions and resisting censorship attempts becomes the dominant strategy for block builders. This approach not only addresses the challenges of soft censorship but also mitigates the risks of hard censorship. It has the potential to improve the integrity of ethereum significantly.

This proposal is not meant to replace inclusion lists but should be considered a progressive move towards more robust forms of censorship resistance.

## What motivates validators

Under mev-boost, the most profitable block always wins; this assumes that all actors in the PBS auction are rational. However, it fails to account for the fact that some validators are motivated by factors other than profit. PBS has impacted censorship resistance and pushed the network towards centralisation. Many validators (over 8% at last count) refrain from using mev-boost, perhaps because of concerns around censorship and centralisation. Validators may perceive themselves as being more ethereum aligned if they are not running mev-boost because they accept transactions that block builders censor. 

## Block Builders

The current implementation of PBS is designed so that block builders will pay the maximum possible to validators, to the detriment of all other factors. One reason PBS was introduced was the idea that block builders would be better at building blocks than validators, especially solo validators. They would be better connected and have access to private order flow. The problem is that better was assumed to mean more profitable. The current optimum strategy is to be a vertically integrated searcher builder that can pay the most for a block. Anything that gets included in that block is secondary and only there to reduce the cost burden on the purchase of the block.

We need to change the game, so the optimum strategy is to fill the block with as many transactions and blobs as possible within the protocol's constraints and pay the validator for the privilege. Put another way, block builders should be better at building blocks than validators. To achieve this new optimum strategy, they must be well connected to the far reaches of the network, accept transactions from any source, and include all valid transactions.

In short we need to force the transaction supply chain to be more representative of validators wishes.


<div style="text-align:center;">
  <a href="/assets/img/blog/block-proposer-inclusion-lists/censorship-pics.png">
    <img src="/assets/img/blog/block-proposer-inclusion-lists/censorship-pics.png" alt="chart showing 10% validators censor - 39% builders censor">
  </a>
</div>

Source: <a href="https://censorship.pics/">https://censorship.pics/</a>

## How to introduce a dilemma

Currently, block proposers are missing out on priority fee rewards. Let's assume an OFAC censored transaction is in the mempool tc, and a block builder creates a 'censored' block bc. The total amount of rewards available is bc + tc. However, the block proposer only gets bc. The block proposer is choosing to accept bc because the rewards are greater than what they could build themselves (see [builder_boost_factor](https://ethereum.github.io/beacon-APIs/#/Validator/produceBlockV3)). The block builder knows it can build a better block than the block proposer as they have more order flow. The only thing that keeps the block builder from bidding just above the fees in a block containing mempool transactions is healthy competition between block builders. 

## Proposer inclusion lists

To introduce some form of dilemma for the block builder, the block proposer should have better visibility of what they are delegating. Currently, the relay only exposes information about fee rewards and gas used. We could (extend builder_boost_factor)[] to include an opinion about gas used that would only require changes on the consensus layer client. But let's go much further; the relay should publish the transaction hashes of the winning bid. The consensys layer client can then check against its view of the mempool and decide if it wants to delegate block building to that relay. 

## Scenarios

Let's assume we have two transactions:

t1 a censored public mempool transaction paying one gwei in tips
t2 a private transaction paying two gwei in tips and two gwei in mev

Builder censors and proposer censors
The block will contain t2. The proposer will receive two gwei in tips, and the block builder will receive two gwei in mev.

**Win-Loose**

Builder censors and proposer accepts
The proposer will build a block and receive one gwei in tips. 

**Loose-Loose**

Builder accepts, and proposer accepts
The block will contain t1 and t2. The proposer will receive three gwei in tips, and the block builder will receive two gwei in mev.

**Win-Win**

Builder accepts, and proposer censors/doesn't enforce
The block will contain t1 and t2 because it is more profitable. The proposer will receive three gwei in tips, and the block builder will receive two gwei in mev.

**Win-Win**

## Considerations

Not all validators would need to implement this initially. The threat of not accepting a block builder's block should be enough incentive for them to choose not to censor transactions. However, which validators censor and which do not will become apparent over time. This may mean that block builders will change their tactics depending on who is proposing the next block.

Smaller transactions and participants are less likely to be sidelined by profit-maximising behaviour. This will lead to priority fees trending downwards.

Block builders will help strengthen the network rather than centralise it.


