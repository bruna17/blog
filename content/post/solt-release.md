---
title: "Introducing 'solt' - The Solidity Tool"
date: 2020-10-30T13:20:38+10:00
draft: false
tags: ["solidity", "dev-tools", "projects"]
---

I'm pleased to announce the [latest release of solt](https://github.com/hjubb/solt/releases/tag/v0.4.0) - the Solidity tool.
Formerly `solc-sjw`, the project quickly outgrew the basic usage of writing Solidity standard json output for your smart contracts
and has been renamed as a result
with a more streamlined set of commands and options to suit all your Solidity contract verification from the command line,
no matter the project's structure or framework you're using to develop with.
<!--more-->

## Installing the latest version (v0.4.0 at time of writing)
These installation steps are available in the README of [the repository](https://github.com/hjubb/solt) which will have
the latest versions and kept up to date, but are replicated here for those wanting to play along at home.
*(Although I highly encourage you to follow the repository for notifications on future releases)*
### Linux
```bash
wget https://github.com/hjubb/solt/releases/latest/download/solt-linux-x64 -O ~/.local/bin/solt
chmod +x ~/.local/bin/solt
```

### Mac
```bash
wget https://github.com/hjubb/solt/releases/latest/download/solt-mac -O ~/.local/bin/solt
chmod +x ~/.local/bin/solt
```


## `solt write`
Using `solc-sjw`, developers were able to write a JSON file containing the source codes of every smart contract in a base folder,
which could be used with [Etherscan's verification portal](https://etherscan.io/verifyContract). This approach was good
to test drive passable solutions, but fell short of the potential for a tool like this. I set out with a simple goal, to remove the
problem of supplying verified source code for your deployed smart contracts. Using the latest version of `solt`
you are now able to achieve the same result as the previous `solc-sjw` tool with some major improvements.

#### Input can now be either a folder (as before), OR a specific file (as heavily requested)
The previous JSON output writer functionality can be replicated, on either a folder or file by running:
```bash
solt write src/somefolder
# or 
solt write src/somefolder/contract.sol
```
in the base of your smart contract's repo (where you'd usually have a truffle config or a `./src` or `./contracts` folder).
This outputs the normal `solc-input[-name].json` so you can write out multiple JSON files if you are verifying a lot of 
contracts at once. The usual `--npm`, `--no-optimization` and `--runs` flags are available, so you'll need to match 
whatever the optimization specific flags with whatever your config might be, or your build tool's defaults.

If you're deploying separately released contracts through a repo with shared dependencies that might look like this:
```bash
$ solt write src/SystemA/OurToken.sol
# collecting etc...
file written to: solc-input-ourtoken.json

$ solt write src/SystemB/Controller.sol
# collecting etc...
file written to: solc-input-controller.json
```
#### Better error handling (ongoing)
You'll now conveniently be presented with an error if `solt` found non-relative/absolute dependencies, usually resulting
from having NPM imported libraries if you're using something like Truffle. The new file functionality removes the 
headaches developers were having where they would have to manually remove tests, migrations and other unrelated contracts from the output,
and entirely eliminates the need to exclude file extensions if you use .sol or .t.sol files for testing.
Some cryptic errors that resulted in trying not to spew too much stacktrace from the CLI has been fixed up so that you'll
get at least some help to point you in the direction of the problem, although in most cases that was covered by not having the `--npm`
flag or not running `yarn` / `npm install` in the repo before trying to collect those dependencies (it me).

## `solt verify`
The most important and exciting improvement, the reason for the project's name change and expansion in functionality comes
with the second mode of operation, **verifying smart contracts from the command line**. There are tools out there that might
plug into your framework or ecosystem, one of the most notable I've seen for Truffle being [Rosco's truffle-verify-plugin](https://github.com/rkalis/truffle-plugin-verify), 
which itself [recently received an update!](https://twitter.com/RoscoKalis/status/1320764654348619788). Some people however
don't have the option, whether by their choice of development tooling or otherwise. This is where `solt` comes in.

Someone using the tools from [dapp.tools](https://dapp.tools) for instance, in conjunction with something like [dapp-pm](https://github.com/hjubb/dapp-pm)
to manage their dependencies was out of luck until now, and I've tried to make it as easy as possible for users to get their deployed contracts
verified with as little headache as possible. This means **including all the API keys** necessary so that (best case) you shouldn't
need to create, store, and manage your own Etherscan API key if you don't want to, although you can do that as well.

The most basic usage for `solt verify` takes the form of:
```bash
$ solt verify [file] [address] [contract name] -c [compiler version like 'v0.6.12']
```

#### `[file]`
Points  to your `solc-input.json` that was generated in the `solt write` step

#### `[address]`
This is simply your `0x` address of the deployed contract


#### `[contract name]`
This can take a couple of forms:
1. the name of the main smart contract (this is required by Etherscan, might be able to automate in future). Right now
it will search your `solc-input.json` for a key in the `"content": ` section for the first match, so it can be as long or short as you want e.g.
`MyERC` will match a key of `./src/mytoken/MyERC.sol`.
2. the name as before with our own mapping or custom name. This is what you see on Etherscan's code section of a contract under "Contract Name:"
which is useful for remapping the name to the overall purpose of the contract with its dependencies e.g.
`src/mytoken/ERC20Impl.sol:UsefulToken`. If you don't use a custom mapping it'll just take the file name without extension, so
the previous example will show up as `ERC20Impl` on Etherscan.

#### `--compiler / -c`
The compiler flag is the same as Etherscan's web portal and I might be able to extract this from build artifacts,
but at the moment it's easy enough to specify.
Made easier is the fact that whatever you specify here is the first available match, so specifying `-c v0.6` will match `v0.6.12+commit.27d51765`
and `-c v0` (if for some reason you like to live dangerously) will at the time of writing this match `v0.7.5-nightly.2020.10.29+commit.be02db49`

#### `--network`
This simply uses a different Etherscan API endpoint, all of which should be supported the same from my testing as their explorer.
Chances are if you can see a deployed contract on Etherscan's explorer you can probably verify it by the same value in this flag as the subdomain
(if you are verifying on mainnet you can leave this blank)

#### `--infura` and `--etherscan` API Keys
I've packaged API keys with the tool, and I can only see the packaged Etherscan key becoming problematic, so this should rarely need to be specified.
If however you want to use your own keys (so other users don't have to create accounts, generate keys and use them here), you can specify these or 
pass them through some CI process or environment variable in the command line:
```bash
solt verify ... --infura $INFURA_KEY --etherscan $ETHERSCAN_KEY
```
 
## Putting It All Together, a Dapp Example
For simplicity, I won't go into how to get dapp.tools up and running on your machine,
nor will I won't go into much detail about setting up the deployment process for a project using dapp.tools,
as these can vary form being as simple as talking directly to your json-rpc eth node with an account unlocked, to deploying with custom scripts
using other libraries and languages that use the compiled ABIs. I will simply substitute the deployment step with `[run deployment magic here 🧙]`.

#### Project initialisation
You'll want to create and enter the directory your contracts will be created and verified from like some variation of the following:
```bash
$ cd ~/coolprojects
$ mkdir verified && cd verified
```
Following this you'll want to run:
```bash
$ dapp init
```
which will generate your basic project structure, and a Makefile used for building / debugging / deploying your project.

Next, we'll create a simple ERC20 token, because the world can't have enough of them -- but wait, instead of writing out 
an IERC20.sol and implementing it, or specifying some cool `mapping(account=>uint256) balances` and other variables, 
we will just use OpenZeppelin's contracts via a simple dapp-pm command:
```bash
$ dapp-pm openzeppelin/openzeppelin-contracts@v3.2.0 ERC20.sol
```

#### Implementing a token
Now all we need is our own implementation of the token with a cool name, symbol and maybe even mint some, so that they can actually be sent
to other users or traded somewhere.

Dapp tools already made us a file at `./src/Verified.sol` which will be our starting point, the only thing we need to add is the import for
OZ's `ERC20.sol` and a constructor which mints us some tokens to play with:
<script src="https://gist.github.com/hjubb/97865cdb06382d430916c270c5675523.js"></script>

Now we have a contract to deploy, and we can do so with our `[run deployment magic here 🧙]` step.

(For actual dapp tools deployment you can look at [their instructions](https://dapp.tools/dapp/#create) or use [remixd](https://remix-ide.readthedocs.io/en/latest/remixd.html) and deploy via injected Web3)

***NB: you will want to take note of whether optimization took place through compilation with how many runs for this next step***

If you want to follow the same config as me, **I didn't use any optimization**, with compiler version `v0.6.7`. As a result
my `solt write` step will look like:
```bash
$ solt write ./src/Verified.sol --no-optimization
```
which will output a file called `solc-input-verified.json`. Taking note of my deployed address, I can verify my contract (which is deployed to rinkeby)
with the following command:
```bash
$ solt verify solc-input-verified.json 0xD66888E0815B87c19b04Fa9DcD495CAb49928912 Verified -c v0.6.7 --network rinkeby
```
which, after executing and waiting for the result, should return us a `Pass - Verified` at the end of execution, or if you've just run the exact
same command as I wrote above, it will probably tell you it's already been verified...
You can check out the output result for my deployed cool token [here](https://rinkeby.etherscan.io/address/0xd66888e0815b87c19b04fa9dcd495cab49928912#code).


Congratulations you are now the proud owner of 100% of the 10*10^-18 total CoolTokens in existence, don't spend them all in one place!

## Wrapping Up
If you've found `solt` useful reach out to me and let me know on [Twitter @harris_s0n](https://twitter.com/harris_s0n) or by showing your appreciation
on the repository [on GitHub](https://github.com/hjubb/solt). If you have any suggestions for feature improvements don't hesitate to reach out on either platform either by DM, tagging me, or raising an issue!
I'll be looking forward to adding extra features as requested and building the Solidity tool you look forward to using.

If you got this far thanks for your time, I hope I save you some in the future with `solt`!