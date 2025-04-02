---
layout: post
title: "Understanding blob transactions"
date: 2026-04-02 09:00 +0000
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

ETH price: $3,000 USD

For a blob transaction including one blob, here's how the calculation would look:

**Transaction Base Fee:**

Transaction gas costs 21,000 gas, assuming payment to an EOA without call data.

Total transaction base fee = 21,000 gas × 50 Gwei = 1,050,000 Gwei (0.00105 ETH) ≈ $3.15 USD

**Blob Base Fee:**

1 blob × 131,072 gas × 10 Gwei per gas = 1,310,720 Gwei (0.00131072 ETH) ≈ $3.93 USD

**Priority Fee:**

Priority fee = 21,000 gas × 2 Gwei = 42,000 Gwei (0.000042 ETH) ≈ $0.126 USD

**Total Transaction Cost:**

Transaction Base Fee (0.00105 ETH, $3.15 USD) + Blob Base Fee (0.00131072 ETH, $3.93 USD) + Priority Fee (0.000042 ETH, $0.126 USD) ≈ 0.00240272 ETH or approximately $7.206 USD total

### Payment for inclusion

The interesting thing about this is that from the ~$7.206, only $0.126 is available for the entire transaction supply chain. Whilst most L2s do not need to worry too much about fast inclusion, it can impact some things like withdrawals/bridges. The only types of rollups that care about fast inclusion are based rollups. 


## Separate blobs and transactions

The other funny thing about eip-4844 is that blobs and transactions are kind of sent separately. This is to keep blocks light so that it doesn't impact things like timing games with MEV. This isn't entirely true, as we will see. However, from a user perspective, sending a transaction is almost the same.


## How to send a blob transaction

```js
    const tx = {
        to: "<RECIPIENT_ADDRESS>",
        maxFeePerGas: parseUnits("50", "gwei"),
        maxFeePerBlobGas: parseUnits("10", "gwei"),
        maxPriorityFeePerGas: parseUnits("2", "gwei"),
        type: 3, 
        blobs: ["<THE BLOB>"],
        commitments: ["<KZG Commitment>"]
        proofs: ["<KZG Proof>"]
 };

    const signedTx = await wallet.signTransaction(tx);
    // rpc eth_sendRawTransaction

```
From a user perspective, things are relatively familiar; just a couple of additional parameters.

## How to read a blob transactions

When a blob transaction is included in a block on the execution layer, it contains a reference to consensus layer blob data via the `blobVersionedHashes` field. This field is an array of hashes that correspond to the actual blob temporarily stored on the consensus layer.

Each `blobVersionedHash` is a KZG commitment to a blob, and it ensures that the data can be verified later.

## How are blobs propagated

![blob transaction flowchar](/assets/img/blog/understanding-blob-transactions/blob-transaction-flowchart.png)

[blob transaction flowchart](https://mermaid.live/edit#pako:eNp1VN1qE0EUfpVhoKA01qZpfrpIwcYgYtBcaC9KIExmTzZDdmfWmdkmaQh4453i35UiFK98BcG36QvYR_DMzrbZJmavdjbfz_nOOZMF5SoEGlADbzKQHJ4IFmmW9CXBJ2XaCi5SJi15bUATZsj15a_PV28_XF9--ZN_20R2ug539f3b398fSWcGPLNCSdKOBUi7CW93vez7T6StpAFpMrMV3DvoefTXn47oji_ATpWebGJPWSxCZlVe9tWPd-QkVnxCelqlKi_cU3Z2fDZ0Dg0ZxmpIrGbSMJ6XbdWWDI704Pi40w0I2PHA0V-VePe80ux-yWddiBisDgyiiJAkgSRVKvboTrfQXvttu1bKjIGbAGpLL1EWdduo25GRkEAe955h_fn7QMK0x-axYuHp4aOhfni8S7BjLPaaTr_kv65PImWMSFcFKDvGrqYA2nhS20XCiQUFNMihAyNC4EwPFiOc4iAUERi7LBm5GUs_4zseZmWCmyBxjQsfJBQZNXAQ50B2CWd8DL60wq_ksFqU1O-GQcOpQ-O6CMnjLBQyKoLNCpdbUuF1kok4JENgHEeSU_MWToUd-5yTi2jAVZIIm2C7zH_9hxqbz5mxGM7Xyyebdus95JOtzXt5pz_kHLQYzX0SFjEhjSWrmkpjcolOPfj52dMSZhXK5cF-qRGt0EiLkAZWZ1ChCeiEuSNdOME-xRIS6NMAX0OGF5X25RI5eEfPlEpuaFpl0ZgGIxYbPGUphr35M7r9qvGOgW6rTFoa1Kq5Bg0WdEaDaq2516gdVQ-OGrVWo1lttSp0ToNmba9xUF9W6EVutb_Xatb38TmsVg_r9Va9tvwHVje2cQ)

## 🔍 Notes:
- ✅ `eth_sendTransaction` — used by the user to submit the blob tx.
- ⚙️ `engine_newPayloadV4` — execution client sends payload (w/ versioned hashes) to consensus client via **Engine API**.
- 🌐 `blob_sidecar_{fork_digest}` — **new gossip topic** to broadcast blob data.
- 🌐 `block_{fork_digest}` — existing gossip topic for beacon blocks.


## What Happens If a Blob Is Missing When a Validator Proposes a Block?

If a validator proposes a block that includes a KZG commitment (versioned hash) for a blob it doesn't actually have (i.e. the blob has not been propagated or is missing), the block will be considered invalid by the network.


Before proposing a block, the validator must:
 1. Gathers blob-carrying transactions from the execution layer mempool.
 2. These transactions include versioned hashes that refer to blobs.
 3. The consensus client must have access to the full blob data because the validator is expected to:
 •   Include the KZG commitment in the beacon block.
 •   Gossip the corresponding blob sidecar along with the block.

If the blob data isn't available at proposal time, the validator cannot safely include that blob tx in the block.

⸻

What if the Validator Does Propose the Block Anyway?
 •   Other consensus clients will:
 1. Receive the block (via block_{fork_digest} gossip).
 2. See that it includes certain KZG commitments (versioned hashes).
 3. Expect corresponding blob sidecars (via blob_sidecar_{fork_digest} topic).
 4. Attempt to verify the KZG commitment against the blob.

If the blob data is missing or doesn't match the commitment:
 •   Clients reject the block as invalid.
 •   Validator may be slashed or penalized depending on the consensus rules and context.

## Most blocks are built by block builders

Blobs can (and should) be gossiped before the block proposal.
•   The consensus client (CL) will start gossiping the BlobSidecar as soon as:
•   It receives the blob from the execution client via local API.
•   It knows the blob's intended KZG commitment.

This early gossip helps ensure that:
•   All peers have the blob data by the time the block is proposed.
•   The proposer (or any client) doesn't get stuck waiting for blobs after seeing the commitment.


## Blob Overhead Hits Block Builders

The Block builder now has to:
 1. Receive/fetch blob transactions which include the blobs themselves.
 2. ReCompute KZG commitments and proofs.
 3. Package blobs in a separate sidecar.
 4. Propagate them to the network ahead of time.


But block builders hold back the building of the block as long as possible, so it benefits them to send multiple options for blobs. They may receive a more valuable blob transaction late in the block. The blob gossip channel will evolve and be used for optionality (there will be more blobs per slot than what is included). Especially as rollups compete to get in blob inclusion.

## Outsized downside risk

The Block Builder must now be concerned with rejection due to missing blobs. Not only must they ensure that the block has been built correctly with signed commitments, etc., but they must also ensure blob propagation to the proposer and the majority of validators! I think this is great for ethereum as it forces block builders to act as the glue that strengthens network resilience. If you risk losing an entire block's worth of priority fees for an extra $0.126 USD, you will be very conservative when you include that transaction.