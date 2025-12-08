---
layout: post
title: "Post-Fusaka Blob Pricing: Understanding the New Rules"
date: 2025-12-08 09:00 +0000
comments: true
categories: [ethereum, blobs, eip-4844]
---

Fusaka introduced the first meaningful evolution of Ethereum's blob fee mechanics since Dencun. On the surface, nothing dramatic changed. Blobs still have their own fee market, with a base fee that adjusts based on supply and demand. But Fusaka added a second pricing constraint that alters how the *effective* blob price is determined.

## The Key Change: A Blob Base Fee Floor

Before Fusaka, the blob base fee was entirely governed by blob usage. If demand fell, the base fee could drop very low. Fusaka introduced a **floor** on blob pricing tied to the execution layer base fee:

``` js
effectiveBlobBaseFee = max(
    blobBaseFee,
    blobFloor(executionBaseFee)
)
```

Where:

``` js
blobFloor(executionBaseFee) = executionBaseFee / 16
```

This means the effective blob base fee used for payment is whichever number is **higher**: the 1559 style blob market's base fee, or the floor
imposed by the execution base fee.

Fusaka therefore creates a soft coupling between the previous block's execution gas and the blob gas, even though blobs remain a separate fee market.

## Why Add a Floor at All?

Since the Pectra blob prices have basically collapsed to 1 wei, blobs have become artificially cheap relative to execution gas, distorting incentives for how Layer 2s post data.

The floor ensures blob pricing remains within a reasonable band relative to the rest of the network's pricing dynamics.

## What Developers Need to Update

If you're building anything that interacts with blob indexers, explorers, fee estimators, or L2 infrastructure, there are three concrete changes to make.

### 1. Treat the blob base fee as *two* values

Store or compute:

- `blobBaseFee`
- `blobFloor(executionBaseFee)`
- `effectiveBlobBaseFee` (the max of the two)

The effective fee is what users actually pay.

### 2. Don't assume low-demand periods mean cheap blobs

The floor prevents blob prices from dropping below `baseFee / 16`, so
blobs will no longer tail off to near-zero in quiet periods.

### 3. Expect the blob market to be smoother but less independent

Blobs still adjust based on blob demand, but the execution base fee now
sets a lower bound on how low blob prices can go. That makes pricing
more predictable but also more coupled to general network conditions.

### Example

Let's assume a 21,000 gas call with a single blob costing 131,072 blob gas and a fixed 1 gwei prioirty fee and ETH price: $3,000 USD


| Scenario                                      | Exec base fee | Raw blob base fee | Floor = base/16 | Effective blob base fee | Floor active?           | Total Gas Cost (gwei) | Total Blob Cost (gwei) | Total Cost (gwei) | Approx USD ($) |
|-----------------------------------------------|----------------|--------------------|------------------|--------------------------|---------------------------|------------------------|-------------------------|--------------------|----------------|
| **A: No floor**                 | 30             | 5                  | 1.875            | 5                        | No                        | 651,000                | 655,360                 | 1,343,000          | **$4.03**      |
| **B: Floor active**             | 80             | 2                  | 5                | 5                        | **Yes**      | 1,701,000              | 655,360                 | 2,393,000          | **$7.18**      |
| **C: High-demand blobs**        | 40             | 20                 | 2.5              | 20                       | No                        | 861,000                | 2,621,440               | 3,523,440          | **$10.57**     |
| **D: High-demand gas** | 100            | 6                  | 6.25             | 6.25                     | **Yes**      | 2,121,000              | 819,950                 | 3,248,950          | **$9.75**      |

## Summary

Fusaka didn't overhaul the blob market, but it added one crucial rule:

> **Blob fees now have a floor equal to `execution_base_fee / 16`, and
> the effective blob price is the maximum of the raw blob base fee and
> this floor.**
