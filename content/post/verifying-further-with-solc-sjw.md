---
title: "Verify Even More of Your Contracts on Etherscan"
date: 2020-10-23T17:23:24+10:00
draft: false
tags: ["solidity", "dev-tools", "projects"]
---
### Background

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
<!--more-->

### Making it work (we are still here)

The simplest projects were the easiest to cover [at first](https://www.artima.com/intv/simplest.html#part3); only using relative or absolute imports located in a specific folder,
but most sufficiently complex projects quickly outgrow this structure and start importing other libraries for the sake of convenience and/or safety.
There's no need to write your own ERC20 compatible contract from scratch when there are [audited contracts](https://www.github.com/openzeppelin/openzeppelin-contracts) out there that will take care of the hard parts.

The first version of solc-sjw had support for these basic structures or for those using something like [dapp](https://dapp.tools/dapp/)
with a tool like [dapp-pm](https://github.com/hjubb/dapp-pm) to solve the problem of importing versioned and audited contracts from existing sources.
It was obvious even before building this prototype that the majority of people would be using something like truffle with [imports referenced in different ways](https://github.com/hjubb/solc-sjw/issues/3).

**Enter, the --truffle flag**

[Version 0.3.0 of solc-sjw](https://github.com/hjubb/solc-sjw/releases/tag/v0.3.0) now has an optional flag for hunting down truffle or other npm compatible dependencies in the `./node_modules` directory (nothing for globally installed dependencies, yet), 
and I'm glad to report it has worked for an array of test projects I deployed on rinkeby including a project built using Truffle, a project built using dapp.tools (with dapp-pm), and the same dapp.tools project deployed via remix in the browser.

An example usage now looks like the following (for a Truffle project):
```
$ truffle migrate
...
$ solc-sjw -d ./src -no-opt --truffle
```

You'll now have a `solc-input.json` file to upload to Etherscan's [verification site](https://etherscan.io/verifyContract) using the **Solidity (Standard-Json-Input)** compiler type.

### Gotchas and future improvements
Finally, some troubles I had running this across some projects I was testing with. Remix proved to be the easiest way to go through the process as deployment could be wrapped into my metamask provided Web3 instance
and all the compiler options and flags were right in front of me, so I didn't have to hunt down which version of solc my build step was using (this took the majority of my debugging in other projects, and I **HIGHLY RECOMMEND** taking note of your solc flags and if optimization is enabled by your framework or build tool in the contracts you deploy)
 
In the next version I'll look at adding a specific file (and it's dependencies) flag via `-f / --file` so you can use either that, or the existing directory flag `-d / --dir`.
I'll also look at making a separate command to use the `solc-input.json` file generated from the main function of the tool to automate the upload and verification through Etherscan's API.
The 'verify' command might look something like `solc-sjw verify [0x contract address] [solc-input.json file]` with flags for the solc version and an Etherscan API key (or I might ship one with the release so you don't have to register and do all of that yourself, but might get rate limited using the shared one if you can deal with that).

Keep up to date on developments by following the project on [github](https://github.com/hjubb/solc-sjw) and following me on [twitter](https://twitter.com/harris_s0n) to keep up with any updates and future projects I post!