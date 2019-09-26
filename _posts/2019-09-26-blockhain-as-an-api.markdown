---
layout: post
title: "Blockchain as an API"
date: 2019-09-26 08:00:00 +0000
comments: true
categories: ["ethereum", "blockchain", "smart contracts"]
---

## Distributed
Because blockchains are not centrally hosted, they offer a great way to facilitate communication between multiple parties. You can utilise the trusted distributed nature of blockchains to reduce costs and friction between parties. 

## Standards
To achieve inter-party communication, consortium members must agree what an api would look like. It requires standards to be written, published and accepted. The alternative is to have a central authority that dictates terms and charges for membership (e.g. payment gateway providers). 

## Security
Every interaction that changes state on the blockchain needs to be verified and therefore trusted, they can be used as a system of always available asynchronous communication. Furthermore, because of their secure nature, they can be used as a means of dispute resolution. 

## Example
### Tariff Updates
The energy price comparison market has several players that all compete to try and get you to switch your energy provider. To do this perform a price comparison and present you with different tariffs. To get the lists of tariffs, each price comparison company has to chase down all the energy companies to find our their tariffs. They pick up a phone and get the information emailed or faxed over to them; it's then codified into their own systems for their users. It is hugely inefficient and costly for both the energy provider and the comparison site. 

### Switching
Now that you're browsing through the available tariffs you find the deal you want and attempt to switch.  The switching company takes that information and faxes/emails it over to the energy provider. The switching company has not done any KYC and are just passing the data over. The energy company doesn't have to honour the tariff price at this point; it is just a referral. It doesn't happen very often, but mistakes are made (eg someone enters 0.04p kWh rather than 0.05p kWh).

### Public Data
The first thing you can do is make all the data about tariffs publically available. It needs to be available for everyone to see and reference. The crucial difference between a blockchain-based solution and having a central reference data authority is access. The business model for a reference data agency is to accumulate data and charge for access. If it's on-chain, it's freely available, verified and trusted that means people can use it as a source truth about tariff data.

### Onchain Switching
The next you could introduce is on-chain switching. This would allow energy consumers to switch tariffs without the need of using an intermediary. The switching services would be better able to highlight deals that users could switch to. 

## Summary
Using blockchain as a shared api gateway between parties is a great use case. You have guarantees about data integrity and provenance along with a historical record of previous information. It could be used for a wide variety of tasks.