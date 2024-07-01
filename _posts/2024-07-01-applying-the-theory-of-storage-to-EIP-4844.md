---
layout: post
title: "Applying the Theory of Storage to EIP-4844"
date: 2024-07-01 09:00 +0000
comments: true
categories: [ethereum, blobs, eip-4844]
---

> tldr; blobs were in backwardation on June 20th


The recent Blob Contention Event on June 20th, 2024, detailed in a retrospective by Blocknative, highlighted a situation where the cost of blob data temporarily exceeded that of call data, contrary to normal expectations. This article explores the economic theories of contango and backwardation as they apply to EIP-4844 and how these concepts can help us understand and navigate the complexities of Ethereum's data markets. For more details on the event, see the original retrospective [here](https://www.blocknative.com/blog/june-20th-blob-contention-event-retrospective).

# Introduction

The Theory of Storage is a fundamental concept in economics and finance that explains the relationship between the supply and demand of commodities, the costs of holding or storing these commodities, and their future prices. This theory is particularly relevant in markets where physical commodities like oil are traded and stored. This article explores how the Theory of Storage can be applied to EIP-4844, focusing on the dynamics between blobs and call data.

## The Theory of Storage Explained

The Theory of Storage posits that the price of a commodity for future delivery (the futures price) is determined by the current spot price, the cost of carrying the commodity until the delivery date (storage costs), and the convenience yield (the benefits of having physical possession of the commodity).

## Contango and Backwardation

When applied to commodity markets:

	•	Contango occurs when the futures price of a commodity is higher than the spot price, indicating that the market expects the price to rise in the future.  

	•	Backwardation happens when the futures price is lower than the spot price, suggesting that the market expects prices to decline.

The same logic can be applied to describe the term structure of yields in government gilts and treasury bills:

	•	Contango occurs when the yield on a longer-term bill is higher than the yield on a shorter-term bill, indicating that the market expects interest rates to rise.

	•	Backwardation happens when the yield on a shorter-term bill is higher than the yield on a longer-term bill, suggesting that the market expects interest rates to decline.

# Applying the Theory of Storage to EIP-4844

In the context of EIP-4844, we can draw parallels by considering block space a commodity and analyzing the dynamics between the different types of block space.

## Blobs vs. Call Data

## Contango in the Blob Market

Under normal conditions, we expect the blob market to be in contango, meaning blobs should be cheaper than call data. This reflects the goal of EIP-4844, which was to reduce the transaction cost of "blobs" of data.

Backwardation in the Blob Market

However, there may be situations where blobs become more expensive than call data, leading to backwardation. Backwardation could occur due to temporary spikes in demand for blob storage, limited capacity, or increased costs associated with processing blobs. 

## Blob ~~Contention~~ Backwardation

On June 20th, 2024, there was a significant spike in the blob base fee, making blob data more expensive than call data.

In his article, Blair from Blocknative calls this "blob contention." While contention is inherent in block space competition, this event highlights a specific economic imbalance. The inversion is a prime example of backwardation in the blob market. Normally, blobs are expected to be cheaper than call data, reflecting a contango market. However, the costs flipped during this event due to increased transaction volume and insufficient dynamic adjustment by some L2s.

# Conclusion

For more insights about blob backwardation, refer to Blocknative's detailed retrospective on the event.

### References:

https://www.blocknative.com/blog/june-20th-blob-contention-event-retrospective
https://en.wikipedia.org/wiki/Theory_of_storage  
