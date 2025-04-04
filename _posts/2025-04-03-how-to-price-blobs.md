---
layout: post
title: "Understanding blob transactions"
date: 2025-04-03 09:00 +0000
comments: true
categories: [ethereum, blobs, eip-4844]
---
EIP-4844 introduced "blob transactions" almost over a year ago, specifically aimed at significantly enhancing scalability and efficiency for layer-2 solutions like rollups. Understanding how these transactions are priced is essential for developers, users, and anyone interacting with Ethereum's economic system.

### Pricing Components for Blob Transactions

Ethereum blob transactions have three main pricing components:

#### 1. **Blob Base Fee**
- The blob base fee is the fee paid per blob included in a transaction.
- This fee is variable and adjusts according to network demand, similar to the transaction base fee.
- Crucially, blob data is priced separately and more cheaply compared to regular transaction data.

#### 2. **Transaction Base Fee**
- Every Ethereum transaction, including blob transactions, includes a transaction base fee.
- This fee depends on Ethereum's overall network congestion, reflecting the regular cost of block space on the Ethereum mainnet.
- This fee is burned (removed from circulation), maintaining Ethereum's deflationary properties post-EIP-1559.

#### 3. **Priority Fee (Tip)**
- Also known as the miner (or validator) tip, this fee incentivizes validators to include your transaction quickly.
- Unlike the base fees, the priority fee goes directly to validators as an additional reward.

### Example: Calculating Blob Transaction Costs

Let's assume the following example scenario:
- Transaction Base Fee: 50 Gwei
- Blob Base Fee: 10 Gwei per blob
- Priority Fee: 2 Gwei
- ETH price: $3,000 USD

For a blob transaction including one blob, here's how the calculation would look:

**Transaction Base Fee**

Transaction gas costs 21,000 gas, assuming payment to an EOA without call data.

- 21,000 gas × 50 Gwei = 1,050,000 Gwei (0.00105 ETH) ≈ **$3.15 USD**

**Blob Base Fee**

- 1 blob × 131,072 gas × 10 Gwei = 1,310,720 Gwei (0.00131072 ETH) ≈ **$3.93 USD**

**Priority Fee**

- 21,000 gas × 2 Gwei = 42,000 Gwei (0.000042 ETH) ≈ **$0.126 USD**

**Total Cost**

- 0.00105 ETH + 0.00131072 ETH + 0.000042 ETH = **0.00240272 ETH**
- ≈ **$7.206 USD total**

### Payment for Inclusion

From the ~$7.206, only $0.126 is available for the transaction supply chain. While most L2s may not worry about fast inclusion, this can impact withdrawals and bridges. Based rollups are particularly sensitive to inclusion time.

## Separate Blobs and Transactions

EIP-4844 allows blobs and transactions to be sent separately to reduce block size and avoid MEV timing games. However, to users, the transaction process remains mostly unchanged.

## How to Send a Blob Transaction

```js
const tx = {
    to: "<RECIPIENT_ADDRESS>",
    maxFeePerGas: parseUnits("50", "gwei"),
    maxFeePerBlobGas: parseUnits("10", "gwei"),
    maxPriorityFeePerGas: parseUnits("2", "gwei"),
    type: 3, 
    sidecar: {
        blobs: ["<THE BLOB>"],
        commitments: ["<KZG Commitment>"],
        proofs: ["<KZG Proof>"]
    }
};

const signedTx = await wallet.signTransaction(tx);
// rpc eth_sendRawTransaction
```

From a user's perspective, it's just a familiar transaction with a couple of added parameters.

## How to Read a Blob Transaction

Blob transactions reference consensus-layer blob data via the `blobVersionedHashes` field. Each entry is a KZG commitment, enabling future data verification.

## How Are Blobs Propagated

![blob transaction flowchart](/assets/img/blog/understanding-blob-transactions/blob-transaction-flowchart.png)

[Blob Transaction Flowchart](https://mermaid.live/edit#pako:eNp1VN1qE0EUfpVhoKA01qZpfrpIwcYgYtBcaC9KIExmTzZDdmfWmdkmaQh4453i35UiFK98BcG36QvYR_DMzrbZJmavdjbfz_nOOZMF5SoEGlADbzKQHJ4IFmmW9CXBJ2XaCi5SJi15bUATZsj15a_PV28_XF9--ZN_20R2ug539f3b398fSWcGPLNCSdKOBUi7CW93vez7T6StpAFpMrMV3DvoefTXn47oji_ATpWebGJPWSxCZlVe9tWPd-QkVnxCelqlKi_cU3Z2fDZ0Dg0ZxmpIrGbSMJ6XbdWWDI704Pi40w0I2PHA0V-VePe80ux-yWddiBisDgyiiJAkgSRVKvboTrfQXvttu1bKjIGbAGpLL1EWdduo25GRkEAe955h_fn7QMK0x-axYuHp4aOhfni8S7BjLPaaTr_kv65PImWMSFcFKDvGrqYA2nhS20XCiQUFNMihAyNC4EwPFiOc4iAUERi7LBm5GUs_4zseZmWCmyBxjQsfJBQZNXAQ50B2CWd8DL60wq_ksFqU1O-GQcOpQ-O6CMnjLBQyKoLNCpdbUuF1kok4JENgHEeSU_MWToUd-5yTi2jAVZIIm2C7zH_9hxqbz5mxGM7Xyyebdus95JOtzXt5pz_kHLQYzX0SFjEhjSWrmkpjcolOPfj52dMSZhXK5cF-qRGt0EiLkAZWZ1ChCeiEuSNdOME-xRIS6NMAX0OGF5X25RI5eEfPlEpuaFpl0ZgGIxYbPGUphr35M7r9qvGOgW6rTFoa1Kq5Bg0WdEaDaq2516gdVQ-OGrVWo1lttSp0ToNmba9xUF9W6EVutb_Xatb38TmsVg_r9Va9tvwHVje2cQ)

## 🔍 Notes

- ✅ `eth_sendTransaction`: used by the user to submit the blob tx.
- ⚙️ `engine_newPayloadV4`: execution client sends payload to consensus client via **Engine API**.
- 🌐 `blob_sidecar_{fork_digest}`: **new gossip topic** for blob data.
- 🌐 `block_{fork_digest}`: existing gossip topic for beacon blocks.

## What Happens If a Blob Is Missing?

If a block is proposed with a missing blob (i.e. the data matching a KZG commitment is absent), it’s **invalid**.

### Validator Duties Before Proposing a Block

1. Gathers blob txs from the mempool.
2. These txs include versioned hashes.
3. Consensus client must have the blob:
   - Include commitment in the block.
   - Gossip the blob sidecar.

If blob data isn’t available, the transaction must be excluded.

### What if the Validator Proposes It Anyway?

Consensus clients will:

1. Receive the block.
2. Detect the commitments.
3. Expect blob sidecars.
4. Attempt to verify commitments.

If verification fails:
- The block is rejected.
- Validator risks slashing or penalties.

## Most Blocks Are Built by Builders

Blobs should be gossiped **before** block proposal:

- Consensus client starts gossiping BlobSidecar after:
  - Receiving blob via local API.
  - Knowing its intended commitment.

This helps:
- Ensure blob availability.
- Avoid proposer delays.

## Blob Overhead for Builders

Builders must:

1. Fetch blob txs with blobs.
2. Recompute KZG commitments/proofs.
3. Package blobs separately.
4. Propagate ahead of block proposal.

Builders may hold block construction late to choose better blobs — adding competition among rollups for inclusion.

## Risk for Builders

If blobs are missing, an entire block could be rejected. Builders have every incentive to ensure complete blob propagation and correctness — especially for just **$0.126 USD** in priority fees. This forces builders to strengthen network reliability.

# Summary

You should probably pay more for blob transactions.
