---
layout: post
title: "PropAMMs Revisited: Curves, Latency and the Strange Resilience of Passive Liquidity"
date: 2026-07-09 09:00 +0000
comments: true
categories: [ethereum, dex, propamm]
---

In my previous post on PropAMMs, I framed them as the next step in Ethereum’s trading-primitive evolution: a move from passive liquidity curves to programmable market makers, where pricing intelligence sits closer to execution.

After listening to [a deeper discussion](https://x.com/apriori0x/status/2071965212387754341) between [aPriori](https://x.com/apriori0x) and [Haikane](https://x.com/haikane), a couple more things clicked. PropAMMs are not simply AMMs with faster prices. They are a different way of managing market-maker risk.

To state the obvious, PropAMMs update curves, not quotes. They do not merely publish an oracle price, nor do they continuously sign firm prices for individual takers. They publish parameters for a custom pricing function which, once included onchain, behaves like an AMM.

It changes how inventory risk is expressed and, perhaps most interestingly for Ethereum, where coordination happens.

## It's all about the curve, stupid!

The easiest mistake is to think of a PropAMM as an iteration on an RFQ system.

It's not; with an RFQ a market maker signs a price that's available for some period of time, and the trader has the option to execute against that price. If the market moves before settlement, the quote can become toxic: the taker exercises when the price is favourable to them and ignores it when it is not.

This is a familiar form of adverse selection. The market maker has written a short-lived option and given it to the taker for free.

A PropAMM works differently.

The market maker does not stream guaranteed executable quotes. It streams curve parameters to a venue. Any displayed “quote” is only an indication of what a trade would receive against the current state, much like the price shown for a Uniswap v2 pool. Once the parameters land onchain, the pool behaves like an AMM. If somebody trades before you, the state changes and your execution price changes with it. If somebody trades in the opposite direction, you may even receive positive slippage.

Whilst it might sound like a technicality, it fundamentally changes the risk model.

Imagine a market maker sends the same signed RFQ quote to five takers. Until settlement, the market maker does not know whether none, one or all five will execute, so they don’t know their final inventory. The quote therefore has to price not only the proposed trade, but the possibility that multiple stale quotes are exercised at once. The rational response is to widen the spread.

A market maker can try to overquote and honour only the first fill, but that creates a different problem: uncertain execution for takers, particularly during volatile periods when reliable liquidity matters most.

A PropAMM handles the same problem through state. If five traders execute in the same direction, the first receives the best price, the second moves further along the curve, and so on. Each trade changes the price available to the next trader. Inventory risk is expressed continuously through the curve rather than approximated through quote expiries and fill limits. The curve is programmatically acting as an inventory-control mechanism.

## Stateful execution is the point

Traditional AMMs have always been stateful. Their apparent simplicity hides an important property: every trade leaves information behind. Reserves change, the marginal price changes and the next trader faces a different market.

PropAMMs retain that property while allowing the shape and position of the curve to be informed by sophisticated offchain pricing models.

Offchain systems can estimate fair value, volatility, inventory targets and short-term flow. The onchain curve then governs how that view is exposed to the market and how subsequent trades alter the available liquidity.

You can think of it as a market maker publishing a set of rules rather than a list of promises.

The rules may say, in effect: “Around this reference price I am willing to provide liquidity, but as my inventory moves away from target, the marginal price should become progressively less attractive.” The precise curve can encode far more nuance than a constant-product AMM while still preserving deterministic, shared execution.

That combination is what makes PropAMMs interesting. They import some of the intelligence of professional market making without discarding the statefulness and composability of an AMM.


## Latency is inventory risk

PropAMMs do not eliminate stale-price risk. The longer the gap between computing curve parameters and settling a trade, the more the external market can move and the more onchain activity can occur before those parameters are used. For a professional market maker, that uncertainty ultimately becomes inventory risk.

On Solana, you can update curves frequently. Fast blocks and relatively cheap state updates make it practical to keep refreshing the onchain representation of the market maker’s view.

Ethereum has a different cost structure. L1 blocks are slower, and state updates are expensive. Even on “fast” L2s, the relevant question is not simply the advertised block time. What matters is how close the market maker can get its latest parameters to the point of execution, and what guarantees exist around ordering and inclusion.

That pushes PropAMM design towards the source of order flow.

Rather than publishing a curve update and waiting for it to propagate, a market maker can send updated parameters directly into the block-building or sequencing pipeline. The builder or sequencer serves as the bridge between off-chain pricing and on-chain settlement.

This is where apparently basic builder features become economically meaningful. On Ethereum, we are seeing builder support for conditional execution patterns in which curve updates are placed near the top of the block but are streamed continuously, with the last best-effort update being included. On an L2, a centralised sequencer can make this operationally straightforward.

It reduces the time between the market maker’s latest view and the state against which the user trades. Instead of paying for a standalone onchain update every time the model changes, the update can be carried alongside economically relevant order flow.

Of course, this introduces a dependency on builders and sequencers. The market maker is no longer relying only on the smart contract; it is also relying on an execution pipeline to deliver freshness and ordering. 


## All liquidity is created equal, but some liquidity is more equal than others

Liquidity has a shape and freshness. A large passive AMM position may look deep but become expensive when volatility rises or arbitrageurs move the pool away from the external market. An RFQ may look precise but carry fill uncertainty, expiry risk or information leakage. A PropAMM curve may offer excellent execution near its centre while becoming deliberately defensive as inventory accumulates.

The relevant question is not simply, “How much TVL does this venue have?” It is, “What execution can this liquidity provide, under what market conditions, and how reliably?”

We’re seeing a genuinely novel primitive emerge. I’m excited to see where this goes next.

