---
layout: post
title: "Lies, damned lies, and simulations"
date: 2024-09-23 09:00 +0000
comments: true
categories: [ethereum, mempool, EIP-7702, EIP-3074, batch-transactions]
---

Protecting users from "drainers" has long been a challenge on Ethereum. These contracts trick users into signing harmful transactions, often disguised as simple token swaps. While wallets rely on transaction simulations to combat these attacks, simulations frequently fall short in preventing fund loss. In this post we will explore how drainers can bypass simulations and discuss strategies to mitigate these risks.

Please note, I am not a security expert!

### Block Number Tricks

Let's look at a simple contract that returns odd or even depending on the block number it executes on

```solidity
contract FlipContract {

    function Flip() external view returns (string memory) {
        if(block.number % 2 == 0) {   
            return "even";
        } else {
            return "odd";
        }
    }
}
```

Let's say we want to simulate this, at block 10 we get even. Then we send it to the public mempool and we get included in block 11, the result is odd. The contract code can change based on the state of the block. Let's look at what drainers could do to trick a simulator.


### Coinbase Tricks

One trick drainers use is to detect something else in the block so as to trick the simulator. When simulating, the block number and other fields are set as defaults. The coinbase is set to the miner address of the simulator you're running on, so if you leak information about your simulator's coinbase, your simulations can be compromised.

```solidity
contract CoinbaseContract {

    function Claim() external view returns (string memory) {
        if(block.coinbase != 0xMySimulator) { 
            revert("Are you living in a computer simulation?!")
        } else {
            return "There are some problems that technology can't solve";
        }
    }
}
```

The other option is to target the miner making the block. On mainnet, with over 80% of transactions being built by just two parties, it's easy to target something specific for those builders ([mevboost.pics](https://mevboost.pics/)). The same applies to L2s, especially those with a single sequence (they'll 100% decentralise soon, I'm sure)

```solidity
contract CoinbaseContract {

    function Claim() external view returns (string memory) {
        if(block.coinbase == 0x95222290DD7278Aa3Ddd389Cc1E1d165CC4BAfe5) { // beaver build
            revert("Damn you!")
        } else {
            return "Dam, son!";
        }
    }
}
```

Another trick is to use the block timestamp. 

```solidity
contract TimestampContract {

   function Claim(uint256 timestamp) external view returns (string memory) {
       
        if(timestamp > block.timestamp) {
            revert("Don't take fame")
        } else {
            return "Don't take money"
        }
    }
}
```

With this attack, the attacker must get the user to sign the transaction with a timestamp in the method call. So, you need to engineer the user to sign something with a large number. The confirmation window on your wallet should show you this, so it might raise suspicion at this point. But the rough idea is that you will simulate the next block, and the timestamp will be 12 seconds in the future. So all the drainer needs to do is pass a timestamp that's over 12s in the future. Things are getting less atomic for the attacker at this point.

### Proxy/Upgradable Contracts

Proxy contracts enable upgradeable smart contracts by acting as intermediaries that delegate calls to the implementation contract. You can't guarantee anything if you simulate against a proxy contract. An attacker could easily front-run with an upgrade and completely change the contract. When you simulate against a proxy, all bets are off!


## Simulating Intents/Meta Transactions

With meta transactions, users sign something in their wallet and return that signed data to the dApp for them to handle. The wallet can simulate what would happen when the signed data is called against the verifying contract. EIP-712 provides a standardised way to prevent cross-contract and cross-chain replay attacks. However, all of the other simulation tricks above are still possible. The main downside to meta transactions is that you are handing control over to the dApp to get it landed on-chain. You, therefore, lose control over when that happens. 

### Other Attacks

I'm sure there are a bunch of other tricks I'm not aware of! To be clear, I am not a security reserchoor just a lowly code wrangler! 

## Simulator Mitigation Strategies

### Next Block Inclusion

You can aim for the next block to minimise the time between simulation and inclusion. As we've seen, many of the techniques use some form of time delay. By dialling your priority fee up to 11 (at the time of writing, a 2 gwei priority fee should get you fast inclusion), you reduce the chances of having a different simulation to execution. Note, this is just a mitigation strategy; you're unlikely to get top-of-block inclusion, so your simulation can always differ even if you get the next block inclusion.

### Conditional Transactions

One option would be to utilise some conditionality about inclusion. You could simulate a transaction at block 11 and then indicate that the transaction is only valid at block 11. There are multiple ways to do this now, with different trade-offs.

#### eth_sendRawTransactionConditional

Currently I belive only [Arbitrum](https://forum.arbitrum.foundation/t/aip-2-activate-support-for-account-abstraction-endpoint-on-one-and-nova/14790) supports this `eth_sendRawTransactionConditional`. But the long and short of it is that you specify the block number in the request. The main downside is that you need to trust the sequencer, which you're doing anyway so YOLO! 

#### eth_sendBundle

On mainnet, traders send the transactions directly to a block builder and specify the `blockNumber` [eth_sendBundle](https://docs.titanbuilder.xyz/api/eth_sendbundle). You need to trust the block builder you send it to. However, the market dynamics are more interesting than a centralised sequencer. The block builder's entire business is built on trust, and if they lose that trust, you can just switch to a different one.

Another alternative is to use a more retail-focused offering like MetaMask [Smart Transacitons](https://metamask.io/news/latest/introducing-smart-transactions/) or [Flashbots Protect](https://protect.flashbots.net/). These do not allow you to specify a block number, but they do time out, so you have a time frame within which you need to simulate. 

## Future Transaction Posibilities

One thing I've been thinking about is how simulations will be impacted by ~~[eip-3074](https://eips.ethereum.org/EIPS/eip-3074)~~ and [eip-7702](https://eips.ethereum.org/EIPS/eip-7702) and what it could enable. 

tldr; It enables an EOA to authorise a contract to make transactions on its behalf, effectively allowing for features like sponsored transactions and batch operations. An EOA can have smart contract-like functionality without deploying a smart contract wallet. Check out [the delegation toolkit](https://metamask.io/developer/delegation-toolkit/) for what possibilities it can enable. 

The thing that really interests me is batching. Most people focus on batching because you can do aprove and swap in a single transaction. But batching enables some really cool stuff that I think will protect users and provide stronger simulation guarentees.

### Batch Preconditions

With the introduction eip-7702 it means that an EOA can potentially use a [multicall](https://github.com/mds1/multicall) contract without downsides. We can then construct a transaction containing our intent, insert our precondition for checking the block, and send it to the multicall to execute the on-chain. It comes at the cost of extra gas, and you will still pay if it reverts. But you could see a situation where you simulate three blocks into the future (a simulation for each block) and put a reasonable expiry on it. This would need to be implemented at the wallet level, or you could imagine a [snap/plugin](https://metamask.io/snaps/) that allows you to do it.  

#### Block Number Protector

```solidity
contract BlockNumberProtector {
    function protect(uint256 validUntil) external {
        require(block.number <= validUntil, "Block number mismatch");
    }
}
```

#### Trace Call Protector

Imagine you could do an onchain call trace like you can with geth https://geth.ethereum.org/docs/developers/evm-tracing/built-in-tracers#call-tracer. You could have a fairly simple allow/block list of contracts you're willing to work with. You might end up with an onchain allow list that's maintained by different security audit firms. The user could then choose what level of security they are willing to accept and just revert anything that interacts with something they're not happy about. Even without a trace call you could have an entry level allow/block list for contracts. https://blog.openzeppelin.com/ironblocks-onchain-firewall-audit looks to be something like this.


### Batch Postconditions

One thing that I find very interesting about batch transactions is the possibility of having post transaction logic. Imagine you had a swap that you wanted to execute, but there's only liquidity on some AMM you've never heard of. You could backrun your trade with a postcondition to ensure you get what you expected. 

#### Balance Checker


```solidity
contract BalanceChecker {

    function assertBalances(
        address[] calldata tokenAddresses,
        uint256[] calldata expectedBalances,
        uint256[] calldata slippageBps
    ) external view {
        require(
            tokenAddresses.length == expectedBalances.length &&
            tokenAddresses.length == slippageBps.length,
            "Array lengths mismatch"
        );

        for (uint256 i = 0; i < tokenAddresses.length; i++) {
            IERC20 token = IERC20(tokenAddresses[i]);
            uint256 actualBalance = token.balanceOf(address(this));
            uint256 expectedBalance = expectedBalances[i];
            uint256 slippage = slippageBps[i];

            uint256 minBalance = expectedBalance - ((expectedBalance * slippage) / 10000);
            uint256 maxBalance = expectedBalance + ((expectedBalance * slippage) / 10000);

            require(
                actualBalance >= minBalance && actualBalance <= maxBalance,
                "Balance outside allowed slippage"
            );
        }
    }
}
```

#### State Simulation Checker

You could go even further and have some form of post state checker.

```solidity
contract StorageValidator {

    struct SlotValue {
        uint256 slot;
        bytes32 expectedValue;
    }

    function validateStorage(SlotValue[] memory slots) public view {
        for (uint256 i = 0; i < slots.length; i++) {
            bytes32 actualValue;
            uint256 slotIndex = slots[i].slot;

            assembly {
                actualValue := sload(slotIndex)
            }

            require(
                actualValue == slots[i].expectedValue,
                "Storage validation failed."
            );
        }
    }
}
```

 The problem with this is that you would be checking for exact storage values! It would probably fail a lot, but if you wanted to be very paranoid it would be kind of cool.



## Conclusion

The golden rule for simulations is it will always be out of date.

**A simulation is only valid for the state at which you simulate against**

Simulations provide some level of protection, they are far from foolproof. To better protect users, new strategies such as next block inclusion, conditional transactions, and the use of batch preconditions and postconditions must be explored and implemented. As Ethereum evolves, it’s crucial to continually adapt and improve our security measures to stay ahead of malicious actors.

#### References

- [EIP-3074](https://eips.ethereum.org/EIPS/eip-3074)
- [EIP-712](https://eips.ethereum.org/EIPS/eip-712)
- [EIP-7702](https://eips.ethereum.org/EIPS/eip-7702)
- [Solidity Security by Sigma Prime](https://blog.sigmaprime.io/solidity-security.html)
- [EVM Solidity Storage Layout by Rare Skills](https://www.rareskills.io/post/evm-solidity-storage-layout)
- [Ironblocks Onchain Firewall Audit by OpenZeppelin](https://blog.openzeppelin.com/ironblocks-onchain-firewall-audit)

