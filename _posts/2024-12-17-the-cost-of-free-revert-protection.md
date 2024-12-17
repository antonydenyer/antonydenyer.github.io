---
layout: post
title: "The Costs of ‘Free’ Revert Protection"
date: 2024-12-17 09:00 +0000
comments: true
categories: [ethereum, mev]
---

In **"Quantifying the Value of Revert Protection"**, Zhu et al. explore how networks offering "free" revert protection shift costs across participants in the transaction supply chain. But who ultimately bears these costs, and why? Let’s break it down.

## Retail Users

When I refer to users, I mean **retail users**—individuals who sign a single type-2 transaction through front-end dApps like [Uniswap](https://uniswap.org/) or [Rarible](https://rarible.com/). Retail-focused services such as [Flashbots Protect](https://protect.flashbots.net/) and [MetaMask Smart Transactions](https://metamask.io/news/latest/introducing-smart-transactions/) offer users **free revert protection**, but how does this work?

When you send a transaction to these services, it gets held in a **private mempool** a centralized database managed by the service provider. This private mempool allows them to **simulate the transaction** at the most recent block, in contrast to traditional wallet simulations that often lag behind. The lag in wallet simulations can result in higher revert rates because they act as a basic sanity check without real-time accuracy.

By simulating transactions on the latest block, these services significantly reduce revert rates for users. However, **revert protection is only guaranteed** at the **block proposer level** - currently that means block builders on L1s and sequencers on L2s.

## Block Builders on L1

L1 transactions typically include **priority fees**, which, at time of writing, with a 1GWei priority fee per gas range between $0.20 and $1 for a swap transaction. These fees represent **value** that can be captured by block builders or other entities in the transaction supply chain.

When a block builder constructs a block, they receive rewards by setting themselves as the **fee recipient**. If a transaction is sent **privately and exclusively** to a single block builder, they can keep this value instead of passing it to the block proposer. To do so, the block builder must account for the cost of including the transaction. These costs include:

1. **Compute Time**: Simulating the transaction before and during block construction.  
2. **Transmission Time**: Sending the block to a relay and the proposer. Larger blocks take longer to transmit.  
3. **Opportunity Cost**: Not including the transaction in favor of higher-value alternatives.

In this case, the block builder can capture the priority fee value (e.g., ~$0.20) or use it strategically to outbid others in a block auction. If, however, **multiple block builders receive the transaction**, competition drives the priority fees to the proposer instead of the block builder.

When transactions are broadcast to **all block builders** builders must assume that their competitors are bidding away priority fees. This eliminates their ability to profit from the transaction directly. Instead, revert protection becomes a **cost of staying competitive**. In short, on L1s, block builders bear the cost of revert protection in a competitive, non-colluding market.

So, how do block builders make money? Hopefully, by offering other ancillary services while remaining credibly neutral. For example, bottom-of-block auctions or other value add services for searchers.

## L2s Are Different

On most L2s, a single sequencer (for now) processes all transactions and collects the gas fees. Unlike L1 block builders, the sequencer **must offer revert protection** because they are the only ones that can.

The paper highlights how revert protection impacts **L2 sequencer revenue**:

- **Retail Users**: Without revert protection, failed transactions often lead to retries, increasing the sequencer’s revenue. With revert protection, the sequencer loses this retry revenue.  
- **Sophisticated Actors**: These actors benefit from revert protection as they can now “max bid” for inclusion without worrying about reverts. This increases fees paid to the sequencer.

The net result? While revenue from retail users decreases, the gains from sophisticated actors outweigh the losses, leading to higher overall revenue for the L2 sequencer.

## Cui Bono?

Revert protection represents a **win-win**. Retail users now access tools—like revert protection that have been available to sophisticated actors for years. This levels the playing field for transaction inclusion.

Free revert protection isn’t free — it redistributes costs across the network. While retail users benefit directly, L1 block builders and L2 sequencers adapt their revenue strategies to account for these costs.  

## References

1. Zhu, B. Z., Wan, X., Moallemi, C. C., Robinson, D., & Bachu, B. [Quantifying the Value of Revert Protection (2024)](https://arxiv.org/abs/2410.19106).
2. Flashbots. [Flashbots Protect](https://protect.flashbots.net/).  
3. MetaMask. [Introducing Smart Transactions](https://metamask.io/news/latest/introducing-smart-transactions/).  
