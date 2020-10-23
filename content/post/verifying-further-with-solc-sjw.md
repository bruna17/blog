---
title: "Verify Even More of Your Contracts on Etherscan"
date: 2020-10-23T17:23:24+10:00
draft: false
tags: ["solidity", "dev-tools", "projects"]
---

This is somewhat of a followup to my good and personal friend [Kendrick's post](https://kndrck.co/posts/verify-contracts-etherscan-100/)
about a tool I am building to help Solidity developers verify their contracts on Etherscan in a non-opinionated or framework-specific way, 
so much of that background won't be covered again. 


A quick summary in the form of a hypothetical project's README might look like this:
```
### Building and testing:
run truffle test / make test / brownie test
### Deploying:
run truffle migrate / make deploy / brownie run deploy / one click via remix

### Verifying deployed contracts:
(TODO figure this out and document)
```

### Making it work (we are still here)
The simplest projects were the easiest to cover; only using relative or absolute imports located in a specific folder,
but most sufficiently complex projects quickly outgrow this structure and start importing other libraries for the sake of convenience and/or safety,
there's no need to write your own ERC20 compatible contract from scratch when there are audited contracts out there that will take care of the hard parts.

The first version of solc-sjw had support for the most basic structure of projects for those using something like [dapp](https://dapp.tools/dapp/)
with a tool like [dapp-pm](https://github.com/hjubb/dapp-pm) to solve the problem of importing versioned and audited contracts from existing sources.
It was obvious even before building this prototype that the majority of people would be using something like truffle with [imports referenced in different ways](https://github.com/hjubb/solc-sjw/issues/3).

### The --truffle flag
Version 0.3.0 of solc-sjw now has an optional flag for hunting down truffle dependencies in 