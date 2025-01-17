---
layout: post
title: "Signal, Feedback, and Protocol Design"
date: 2025-01-17 09:00 +0000
comments: true
categories: [ethereum, protocol design]
---

Feedback mechanisms are the backbone of resilient, scalable systems. In distributed systems, active monitoring and backpressure ensures the system's overall stability. Similarly, horizontal scalability relies on dynamic monitoring to provision resources in response to demand. Dynamic auto-scaling and spot pricing are all standard these days. It wasn't always the case; historically, you'd have to provision servers upfront and then add to them as demand increased. Your ability to scale would be a step function. There are a couple of exceptions, but this is where ethereum is today, the only way to change something is with a hard fork aka a step function. Just take a look at gas limit 

![gas limit over time](/assets/img/blog/signal-feedback-and-protocol-design/gas-limit.png)

[https://etherscan.io/chart/gaslimit](https://etherscan.io/chart/gaslimit)


## Protocol Feedback in Action

There are two meaningful examples of protocol design that embodies this thesis; difficulty adjustment and base fee. These mechanisms enable self-regulation by allowing the network protocol to respond dynamically to changing conditions.

### Difficulty Adjustment 

Bitcoin’s difficulty adjustment acts as a feedback mechanism that ensures block times stay close to the target of 10 minutes, adapting to changes in the network’s hash rate. Roughly every two weeks, the difficulty is recalibrated based on the time it took to mine the previous 2016 blocks. If blocks were mined more quickly than expected, indicating a rise in hash rate, the system increases the difficulty. If blocks were mined more slowly, suggesting a decrease in hash rate, the difficulty is lowered.

This feedback loop operates as a signal mechanism for miners and the network at large. The adjustment ensures the block production rate remains stable, even as the collective computational power of the network fluctuates. 

The simplicity of this feedback and signal system is what makes it so effective. It requires no centralized control, yet keeps the Bitcoin network running smoothly and predictably. This decentralized adjustment is fundamental to Bitcoin’s ability to self-regulate, creating a stable and secure monetary system over time without a hard fork.

### Dynamic Base Fee Adjustment 

Ethereum’s EIP-1559 introduced a similarly impactful feedback mechanism for managing transaction fees. By dynamically adjusting the base fee based on network demand, EIP-1559 reduces gas price volatility, enhances user experience, and optimizes network usage. This mechanism operates autonomously, without the need for social coordination or hard forks to update the base fee. Instead, it uses block utilization as the input for feedback, making fee adjustments seamless and automatic. The design of the algorithm is driven by the principles of dynamic feedback: when demand for block space increases, the base fee rises, effectively raising the cost of transactions and reducing demand. While the algorithm’s design can still be refined, it represents a clear example of how decentralized networks can use feedback loops to self-regulate and balance network resources efficiently.


## The Gas Limit Debate

### Why the Gas Limit Exists
Ethereum's gas limit isn't arbitrary; it's a safeguard. It caps the amount of computational work validators must process within a block, ensuring the network remains decentralized and all validators can operate on reasonably accessible hardware and internet connections. Blocks could grow too large without this limit, making block validation times unmanageable and increasing the risk of validator centralization.

However, the gas limit is not static. Validators can signal preferences to raise or lower it by setting a gas limit when proposing a block. This signalling mechanism provides some feedback but occurs infrequently only when a validator proposes a block. This infrequency makes it difficult to gauge real-time consensus across the entire validator set. If you wanted to wait for the majority of validators to buy in before changing the gas limit, you'd likely need to wait something like 3-6 months.

### Risks of Increasing the Gas Limit Too Much
Raising the gas limit, while tempting for scalability, has well-documented risks. Larger blocks increase the computational load on validators, leading to:  
- **Centralization Risks:** Validators with less powerful hardware may drop out, consolidating power among those with greater resources.  
- **Network Instability:** Block propagation delays could increase fork rates, and validators might miss attestations if they struggle to process larger blocks.  
- **Historical Precedents:** Ethereum has seen bugs and instability during periods of high computational stress, such as the excessive block size increases during gas price surges.

These risks underscore the importance of carefully calibrated gas limit adjustments based on reliable feedback.

### Crypto Twitter as a Social Feedback Mechanism
In the absence of robust technical feedback mechanisms, gas limit debates often spill into **Crypto Twitter**. These debates highlight the community’s differing priorities:  
- **Scalability Advocates:** Argue for higher gas limits to increase throughput and reduce transaction fees.  
- **Decentralization Advocates:** Caution that increasing gas limits risks validator centralization and long-term network health.  

These debates are not a productive substitute for built-in, formal feedback systems. Social feedback is noisy, contentious, and difficult to aggregate into actionable protocol changes. It often descends into a bun-fight based on false assumptions, dodgy data and back of a fag packet math.


## Toward Better Feedback Mechanisms

Ethereum already has structures that could be adapted to improve feedback on parameters like gas limits. Leveraging these mechanisms could reduce reliance on social debate and create more transparent and actionable signals for protocol adjustments.

### Sync Committees
Sync committees, play a critical role in block finalization. Currently, sync committees share essential information, such as block roots and aggregated signatures. By extending their functionality to include feedback signals—such as validators’ gas limit preferences—sync committees could provide a lightweight, decentralized mechanism for gathering network-wide input more quickly.

### Attestations and Epoch Aggregation
Attestations are already the primary mechanism for validator input in Ethereum’s consensus layer. By adding optional feedback metadata (e.g., preferred gas limits), attestations could be used to collect real-time signals from validators without introducing significant overhead. These signals could then be aggregated over each epoch to produce a network-wide consensus on adjustments.  

### Dedicated Committees 
Another approach could involve creating dedicated feedback committees, elected periodically, to aggregate validator preferences and provide dynamic recommendations. This would formalize feedback into a structured process, reducing reliance on ad hoc proposals or social media debates. This is the approach that FOCIL is taking.

While each of these solutions introduces some complexity, the benefits—better decision-making, more responsive governance, and improved decentralization—outweigh the costs.

## Stability Through Dynamic Feedback

What does a well-designed feedback mechanism look like in Ethereum? I think **Good** means achieving a stable maximum gas limit that dynamically adjusts to network conditions without compromising decentralization, liveness, or security. This vision requires embedding dynamic feedback mechanisms directly into the protocol.

Key metrics that could guide these adjustments include:  
- **Missed Slots:** High rates of missed slots indicate validator stress and could signal the need for a lower gas limit.  
- **Liveness Guarantees:** Monitoring validator participation rates ensures the network remains healthy even under varying gas limits.  
- **Fork Rates:** An increase in fork rates is a clear signal of congestion or disagreement among validators, warranting a recalibration.  

By incorporating these metrics into a decentralized feedback loop, Ethereum can create a system that balances scalability with decentralization, dynamically adapting to demand while preserving network health. 


## Conclusion

Ethereum has already demonstrated the value of feedback with mechanisms like EIP-1559, but the network's approach to gas limit adjustments lags behind. Frankly, I'm sick of the Twitter soap opera and would much rather we focused on more formal, built-in feedback systems.

I think leveraging sync committees is the simplest way forward, but I see the benefit in having dedicated committees. Either way, by embedding dynamic feedback into Ethereum's protocol, the network can scale in a more stable decentralized fashion.


### References

**[On Block Sizes, Gas Limits, and Scalability](https://ethresear.ch/t/on-block-sizes-gas-limits-and-scalability/18444)**:  
   This research paper discusses the evolution of Ethereum's gas limits and block sizes, exploring the trade-offs between scalability and decentralization.  
   

**[A Gas Model for Ethereum 2.0](https://ethresear.ch/t/a-gas-model-for-ethereum-2-0/19989)**:  
   An exploration of Ethereum's gas model with suggestions for improvements in gas cost mechanisms and scalability for Ethereum 2.0. 
   

**[Fork-Choice Enforced Inclusion Lists (FOCIL)](https://ethresear.ch/t/fork-choice-enforced-inclusion-lists-focil-a-simple-committee-based-inclusion-list-proposal/19870)**:  
   This proposal introduces a committee-based approach to transaction inclusion, aiming to improve censorship resistance and neutrality within Ethereum.  