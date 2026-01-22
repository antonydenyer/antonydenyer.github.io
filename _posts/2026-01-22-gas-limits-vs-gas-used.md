---
layout: post
title: "Gas Limit vs Gas Used"
date: 2026-01-22 09:00 +0000
comments: true
categories: [ethereum]
---

It feels like you should be able to deterministically run a transaction locally, count the gas, set that as the `gasLimit`, and end up with 100% gas utilisation every time.

Unfortunately, that’s not how Ethereum works in practice.

The `gasLimit` is a **safety cap**. It limits how much work your transaction is allowed to do, and therefore the maximum you *could* pay. You only pay for **gas actually used**. The limit also matters for network safety (it constrains worst‑case computation) and for nodes/miners/validators deciding whether your transaction is even *eligible* to be included/propegated.

There are two protocol rules that make exact gas limits surprisingly awkward and in many cases impossible:

- The **63/64 rule** from [EIP‑150](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-150.md)
- **Storage refunds** applied at the end of execution from [EIP‑2200](https://eips.ethereum.org/EIPS/eip-2200)

## The 63/64 rule (EIP‑150)

Before EIP‑150, contracts could forward essentially all remaining gas to sub‑calls, which created nasty griefing vectors.

EIP‑150 changed the rules: when you make a call, the EVM only makes **63/64 of the available gas** accessible to the callee. In other words, to give a callee `X` gas, the caller needs **more than `X`** available at the call site.

This is why deeply composable transactions (routers calling other contracts calling other contracts) naturally tend to have headroom in the top‑level gas limit. Even if the underlying work is deterministic, gas *availability per frame* is intentionally constrained.

## Storage refunds

Some operations can earn you a refund (e.g by clearing a storage slot). The important detail is *when* the refund is applied: **refunds are accounted for as the call stack unwinds**, effectively at the end of execution.

That means your transaction must have enough gas to succeed **without relying on the refund**. If a call frame runs out of gas, it aborts immediately.

So even if you’re aiming for an exact limit, the number you need to supply is closer to the *pre‑refund* requirement than the final `gasUsed` you’ll see on a block explorer.

## Practical realities

### Wallet and RPC buffer

Most users don’t care about tight gas limits, they care about transactions landing.

RPC providers and wallets bias towards success. Many implementations of `eth_estimateGas` return a value that’s a little higher than the true minimum, and wallets often add a buffer on top. It’s usually close, but not exact.

### State changes

Even if you *could* compute an exact minimum for “right now”, you’re estimating against a snapshot.

Between estimation and inclusion, state can change (balances, allowances, storage slots, AMM reserves, whatever your call depends on). So in practice you’re simulating “this transaction against the current state”, then guessing how that will map onto a future block.

## Exact gas requirement

Older setups sometimes returned very tight estimates, but if you want true minimums you generally need to do the work yourself.

In [this repo](https://github.com/antonydenyer/exact-gas-examples) I sidestep `eth_estimateGas` and build a simple estimator using `eth_call`:

- pick a gas limit using eth_estimateGas
- simulate
- if it reverts, increase the limit
- binary search until you find the smallest limit that succeeds

Script: [exact-gas.js](https://github.com/antonydenyer/exact-gas-examples/blob/main/scripts/exact-gas.js)

## Demo

We’re going to look at what happens when you try to set exact gas values.

Script used to send transactions: [set-greeting.js](https://github.com/antonydenyer/exact-gas-examples/blob/main/scripts/set-greeting.js)

First we set the greeting string back to an empty string.

```bash
❯ npm run set-greeting ""
Transaction hash: 0x829f47af20d2a35d9d048a630acd94adf37221c485a1edfd07b9bdd61a5f4acd
Transaction confirmed in block 10098660 with status success
❯ npm run get-storage
Raw storage value: 0x0000000000000000000000000000000000000000000000000000000000000000
Length marker: 0x00 = 0 (decimal)
Short string
Length: 0 bytes, Slots used: 1
Content:

```

Now we get an exact estimate using the binary search script.

```bash
❯ npm run exact-gas "Hello World"
Initial gas estimate: 51281
Testing lower bound: 46152
Binary search between 46152 and 51281
Binary search between 48717 and 51281
Binary search between 50000 and 51281
...
Binary search between 50882 and 50891
Binary search between 50887 and 50891
Binary search between 50887 and 50889
Minimum gas limit for successful transaction: 50889
```

The first number is the node’s initial estimate, and it’s **392 gas higher** than the minimum we found.

Now we set the greeting to `"Hello World"` using the gas limit we calculated.

```bash
❯ npm run set-greeting "Hello World" 50889
Transaction hash: 0x4acff0eb3d903a666367eb513c923d73430e314c8220acece2b2b19a25d5211f
Transaction confirmed in block 10098703 with status success
```

On Sepolia Etherscan you can see the transaction used 100% of its gas: [0x4acff0eb…5211f](https://sepolia.etherscan.io/tx/0x4acff0eb3d903a666367eb513c923d73430e314c8220acece2b2b19a25d5211f)

When you check storage, you can see it’s only using one slot.

```bash
❯ npm run get-storage
Raw storage value: 0x48656c6c6f20576f726c64000000000000000000000000000000000000000016
Length marker: 0x16 = 22 (decimal)
Short string
Length: 11 bytes, Slots used: 1
Content:
Hello World
```

If we run the exact gas script again with the same value, the minimum is lower.

```bash
❯ npm run exact-gas "Hello World"
Initial gas estimate: 31597
Testing lower bound: 28437
Binary search between 28437 and 31597
Binary search between 30018 and 31597
...
Binary search between 31408 and 31409
Minimum gas limit for successful transaction: 31409
```

That’s because this time we’re not paying the “first write” costs - we’re largely rewriting the same value.

Now we set a larger value that uses two slots.

```bash
❯ npm run set-greeting "The quick brown fox jumped over the lazy fox"
Transaction hash: 0xe35569c1b63c39e7b59f499e7b46e0d5406b91e6ca7905598a269782eb9afb56
Transaction confirmed in block 10098714 with status success

❯ npm run get-storage
Raw storage value: 0x0000000000000000000000000000000000000000000000000000000000000059
Length marker: 0x59 = 89 (decimal)
Long string
Length: 44 bytes, Slots used: 2
Data starts at slot: 0x290decd9548b62a8d60345a988386fc84ba6bc95484008f6362f93160ef3e563
Reading slot 0: 0x290decd9548b62a8d60345a988386fc84ba6bc95484008f6362f93160ef3e563
Slot 0 data: 0x54686520717569636b2062726f776e20666f78206a756d706564206f76657220
Reading slot 1: 0x290decd9548b62a8d60345a988386fc84ba6bc95484008f6362f93160ef3e564
Slot 1 data: 0x746865206c617a7920666f780000000000000000000000000000000000000000
Content:
The quick brown fox jumped over the lazy fox
```

Now if we estimate the gas required to go back to `"Hello World"` you’ll see the requirement is lower than the initial write. This is because the storage allocation has already been paid for.

```bash
❯ npm run exact-gas "Hello World"

Initial gas estimate: 44450
Testing lower bound: 40005
Binary search between 40005 and 44450
Binary search between 42228 and 44450
...
Binary search between 44387 and 44390
Binary search between 44389 and 44390
Minimum gas limit for successful transaction: 44389
```

So we set the greeting back to `"Hello World"` using our exact limit.

```bash
npm run set-greeting "Hello World" 44389
Transaction hash: 0xbac053594756fc1bc365667d49a626458f72fc8d7b3e012c77edc17407b6a22a
Transaction confirmed in block 10098732 with status success
```

[0xbac05359…b6a22a](https://sepolia.etherscan.io/tx/0xbac053594756fc1bc365667d49a626458f72fc8d7b3e012c77edc17407b6a22a)

This time we only use ~80% of our gas limit (44,389 limit vs 35,512 used). The reason is the **refund for clearing the second storage slot**. We still needed the higher limit to survive execution, but the refund reduces the final `gasUsed` accounting.

```bash
❯ npm run get-storage
Raw storage value: 0x48656c6c6f20576f726c64000000000000000000000000000000000000000016
Length marker: 0x16 = 22 (decimal)
Short string
Length: 11 bytes, Slots used: 1
Content:
Hello World
```

## Conclusion

The instinct to chase 100% gas utilisation is understandable. The EVM is deterministic, so it feels like “exact gas” should be a solved problem.

But even with a binary search that finds the minimum successful limit, you’re still dealing with protocol mechanics that push you away from perfect utilisation:

- **Refunds are applied at the end**, so you must provision enough gas to succeed *before* any refund is credited.
- The **63/64 rule** means gas is intentionally not forwarded 1:1 to sub‑calls, so composable call stacks naturally require headroom at the top.

Add the practical layer — buffered RPC estimates, wallet padding, and state changes between estimation and inclusion — and “tight limits everywhere” stops being a useful goal.

Exact gas is a great teaching tool and a handy diagnostic. But for real users, reliability beats elegance: estimate, add a little buffer, and accept that the chain doesn’t owe you a satisfying 100%.