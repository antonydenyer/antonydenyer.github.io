---
layout: post
title: "Ethereum transaction lifecycle and ecosystem health"
date: 2023-01-23 09:00
comments: true
categories: [Ethereum, mev, mempool]
---

# Before the merge - Proof of Work (PoW)

Before the merge, ethereum was secured using proof of work. To validate transactions and add new blocks to the chain, miners would solve mathematical problems. Miners are in competition with each other to solve the problem; the first miner to solve it gets to add the next block to the chain and receive the block reward. In addition to the block reward, the miner also receives transaction fees for the transactions included in that block. This process aims to ensure that the network is decentralised and not controlled by any single entity. The behaviours required of miners led to several "win-win" outcomes for miners and the ethereum network. What were they?

### Excellent network connectivity

A good miner should have a stable and fast internet connection to ensure that they can communicate with the rest of the network and receive new transactions to be mined. A miner needs to have the visibility of as many block headers as possible, so they can attempt to build upon the most favourable block and create the longest chain. They also want to see as many pending transactions as possible so they can select the most profitable transactions to include. Whilst, before the merge, transaction rewards were small compared to block rewards, they were still good enough to incentivise good behaviour. Miners would receive, on average, 2.5 ETH for each block mined - 2 ETH block reward and ~0.5 ETH in transaction fees. 


### Reliability
 
A good miner should be able to operate continuously and reliably without experiencing frequent downtime or technical issues, benefiting the network and the miner.

### Distributed/GeoLocation

A miner may have an advantage by being better connected. This would mean having several geo-located miners that provide good worldwide coverage and connectivity. Again, benefiting the network's breadth and the miner's transaction rewards.

### MEV

Before the merge, many miners participated in off-chain revenue-maximising activities, such as running mev-geth (flashbots) or offering private order flow. Unfortunately, this situation has not improved post the merge; if anything, it's gotten worse.

### Fast block production

When a miner finds the solution to the problem, they have a strong incentive to propose a block as quickly as possible and to disseminate that information as widely as possible. Someone else has possibly found the solution, so they want to get there first. When multiple parties have the same solution simultaneously (a fork), the miner with the best network connectivity and the most distributed block header will more likely have their solution accepted by others. 


## Summary 

On the whole, under PoW, miners were extrinsically incentivised to ensure the network's health, whilst some behaviours could be detrimental to network health. Pending transactions and block headers were being propagated, and blocks were being mined. In short, there was a strong incentive to shout from the rooftops that you had created a block.


# After the merge - Proof of Stake (PoS)

Under PoS, instead of miners competing to add new blocks to the blockchain, validators are randomly selected to propose new blocks. This makes the "mining" process more energy efficient; it also reduces the barriers to entry for validators. Anyone with a reasonably stable internet connection, commodity hardware and enough ETH can participate in securing the network. Under PoS, when a validator is selected to propose a block, they take on a privileged position and receive block rewards and transaction fees. In addition to this, they also receive attestation fees when not proposing blocks. However, if a validator is found to be acting maliciously or not following the rules, they risk being "slashed" with different penalties for different infringements. This creates a disincentive for validators to engage in any kind of bad behaviour and further helps to maintain the integrity and security of the network. What makes a good validator?


### Uptime

Unlike running a miner, which can be turned on and off, when running a validator, you have to be committed for the long haul. A good validator will have a high uptime; if it doesn't, it will be punished for missing attestations. Therefore, they need to actively participate in the validation process as much as possible.

### Good validator connectivity

A good validator should have a reasonably good internet connection. But this doesn't need to be that good; a reasonable ADSL broadband should suffice. They need to communicate with the rest of the validators pool to ensure consensus can be reached. Note a validator does not need to talk to non-validators to receive attestation rewards or block rewards. 

### No competition 

Under PoS, there is no competition for who proposes the next block; the block proposer is randomly chosen from the pool of validators. The only criteria required for being selected is if you're in the pool or not (okay, there are some other nuances here, but you're basically in or out).

### Limited Connectivity 

#### Block propagation requirements

With PoS, the validator still needs to publish a block to other validators to ensure they don't get slashed. However, there is little incentive to do it quickly or widely. You only have to publish it to enough validators in a timely manner. So whilst you still want to ensure that your block proposal doesn't end up on an orphaned fork, the chances of this happening are greatly reduced. It means that there is no downward time pressure to publish a new block, which means that you do not have the incentive to be widely connected to other peers.  

#### Poor network health

A possible side effect of not being widely connected to peers is that the network's health is less robust. Validators do not have any incentive to enable peers to download the chain, whereas in PoW they did. They also do not have any incentive to facilitate connectivity to the outer reaches of the network. Connectivity will centralise towards validators rather than away from miners.  

#### Poor transaction visibility

Because the validator may not be widely connected to the ethereum network, they will likely not have good visibility of the mempool. It means that it will take longer for transactions on the edge network to get propagated, with a limited incentive for anyone to do anything about it.

#### Validators use a delegated transaction pool

This means that a validator may decide to outsource mempool curation to a third party. Currently, this is only possible with mev-boost (flashbots). This is having a centralising effect on the mempool; it's also delegating block-building preferences to a third party (e.g. censorship enforcement).

#### Searchers, MEV and the edges of the network

The groups that may want access to the network's edges are searchers and possibly block builders. With searchers, their only desire is to seek out new MEV opportunities and execute them. It may improve the pending transaction propagation if it is of value to the searcher. However, if it is not, the pending transaction will remain on the network's edge. Searchers do not improve transaction propagation. 

#### Block builders to the rescue?

Block builders are responsible for assembling a block proposal and making it available for validators to use; validators may use them to increase their transaction rewards. They have an oversized privileged position in the network. Currently, there are less than 100 block builders running on the network, with less than ten dominating. With more middlemen, there are more opportunities for censorship and rent-seeking. Currently, block builders don't get directly paid by users submitting transactions to the public mempool. They can take a cut from the transaction fees if they wish, but because of the downward pressure on their fees, a block builder is unlikely to make money on public mempool transactions alone. They only serve as padding to other forms of revenue generation, e.g. searcher bundles. Because of this, there is little scope for block builders to facilitate transaction propagation to other block builders. If anything, they're better off not propagating transactions. We may end up in a situation where blocks are only made up of searcher bundles!

There are other situations where entities may decide not to propagate transactions. When there's a conflict of interest because they're offering some service to searchers, perhaps? 


[https://www.blocknative.com/transaction-distribution-network](https://www.blocknative.com/transaction-distribution-network)

[https://bloxroute.com/products/](https://bloxroute.com/products/)


#### Who will propagate block headers?

The ethereum network is in danger of going the same way as bittorrent. The only people propagating block headers and pending transactions are those with intrinsic motivations for the network's health. There's something wrong with the incentives at the moment. We've replaced the centralising effects of mining pools with a decentralised validator pool and a centralised mempool. The impact of this and the motivations of the actors involved are as yet unclear. 



## References

[https://www.blocknative.com/blog/web3-transaction-lifecycle](https://www.blocknative.com/blog/web3-transaction-lifecycle)

[https://www.paradigm.xyz/2020/08/ethereum-is-a-dark-forest](https://www.paradigm.xyz/2020/08/ethereum-is-a-dark-forest)

[https://ieeexplore.ieee.org/document/9152675](https://ieeexplore.ieee.org/document/9152675)

[https://writings.flashbots.net/the-future-of-mev-is-suave/](https://writings.flashbots.net/the-future-of-mev-is-suave/)

[https://mevboost.pics/](https://mevboost.pics/)
