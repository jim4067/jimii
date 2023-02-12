+++
title = "Week One of EthIndia   "
date = 2023-01-11
category = "Prog"

[taxonomies]
tags = [ "ethereum", "solana"]
+++

I am [part of](https://opensea.io/assets/matic/0xfe8321df99317c365797c4f95c2dbd9beec8c694/8) the EthIndia fellowship 3.0 as a Wei fellow. I'll share a blog of my [overall experience](../eif-experience) and more about the fellowship in a wrap-up blog after my eight weeks are up. I'll use this blog as a weekly log of my experience going through the [speedrunethereum challenges](https://speedrunethereum.com/). It'll also help keep me accountable

In the first four weeks of the fellowship, I'll be going through the [speedrunethereum challenges](https://speedrunethereum.com/) and I will use this blog to posting weekly logs to help keep me accountable.

<!-- more -->

I'll share the challenges I faced while working on the challenges and on Ethereum and how I found the experience coming from a Solana background (yes I am that guy 🤙).

## [challenge zero](https://speedrunethereum.com/challenge/simple-nft-example)

The first challenge is a simple walk-through to get you acquainted with how `speedrunethereum` works and help you set up your dev-tooling.

Some errors I faced working on the first challenge.

0. `Node version` should be the current LTS i.e `v18.14.0`.
1. The first link I got after `deploying` my frontend to [surge.sh](https://surge.sh/) did not work. So I redeployed and the second link worked.

After going through the first challenge, I did some [self-learning about NFTs](https://ethereum.org/en/developers/tutorials/how-to-write-and-deploy-an-nft/), wrote and deployed my contract using [hardhat](https://hardhat.org/). The indexing tools(plus much more) provided by [Alchemy](https://www.alchemy.com/) made the process so so easy. Plus, I didn't know this but, apparently there's a whole [specification](https://docs.soliditylang.org/en/develop/natspec-format.html) on how to write comments Solidity comments.

[Link](https://testnets.opensea.io/assets/goerli/0x41cd30b16d968ee43317289b3b8d96e25872d3bf/3) to NFT I minted (Kobeni Supremacy 🛐).

### Comparison to Solana

Working with NFTs in Solana is different to Ethereum. Lets start from the very beginning.

Firstly, on Solana 'everything is an account.' That's it. Everything you need to know about building on Solana in one simple sentence. On Solana smart contracts are referred to as programs and there is the notion of `native` and `on chain programs`. `native programs` are programs-built into the core of the chain. For example the `system program` address `11111111111111111111111111111111` enables creation of new accounts and transfer of tokens. `on chain programs` on the other hand are deployed to the blockchain by anyone.

So how does this differ with Ethereum you might ask? On Ethereum you have the [ERC721](https://eips.ethereum.org/EIPS/eip-721) specification you have to follows to mint an NFT. That means, implementing your own contracts with all of the specifications or using a battle-tested library like [OpenZeppelin](https://www.openzeppelin.com/). After implementation you HAVE to deploy your contract which is the biggest difference. In Solana there exists a native [token program](https://spl.solana.com/token), address `TokenkegQfeZyiNwAJbNbGKPFXCWuBvf9Ss623VQ5DA`, that you can use to mint NFTs and create tokens. To make it an NFT there's also the metaplex token [metadata program](https://docs.metaplex.com/programs/token-metadata/) to manage metadata.

Secondly, accounts on Solana are stateless. That means your programs/smart-contracts do not store data directly on them. By doing this, some gnarly optimizations such as the [parallelized processing](https://medium.com/solana-labs/sealevel-parallel-processing-thousands-of-smart-contracts-d814b378192)of transactions can be enabled, one of the reasons the network is so fast.. This means that if you are creating a program that has to store some data on-chain you'll have to create another account to do this. Unlike in ethereum where a smart contract which has all of its data in one place.

Finishing it off, there is then the concept of `associated token account` (I'm not talking about this).

## [challenge one](https://speedrunethereum.com/challenge/decentralized-staking)

The second challenge dealt with building a Decentralized staking app.

Fun new things I learned about.

0. `msg` - global variable that exists globally to provide more information about Ethereum. Read more about it in this [blog](https://medium.com/upstate-interactive/what-you-need-to-know-about-msg-global-variables-in-solidity-566f1e83cc69).
1. Testing whether an `event` was emitted is possible as described [here](https://ethereum-waffle.readthedocs.io/en/latest/matchers.html?highlight=events#emitting-events).
2. [Modifiers](https://solidity-by-example.org/function-modifier/) are really cool.
3. emojis and unicode characters are supported in Solidity with the use of the `unicode` keyword.

```js
console.log(unicode'\t 📤 sending to external contract ...');
```

Some modifications I made.

0. Added test to check whether stake event was fired when staking.
1. Used modifiers to hold reusable code like checking whether the deadline was met.

Changed the `execute` function, from this,

```js
function execute() public {
    uint256 contractBalance = address(this).balance;
    //check whether time constraints were met
    uint256 remTime = timeLeft();
    if (remTime != 0) {
      revert('Stake period has not expired yet');
    }
    //check threshold
    require(contractBalance >= THRESHOLD, 'threshold not reached');

    //? send to external contract
    console.log(unicode'\n\t 📤 sending to external contract...\n');
    exampleExternalContract.complete{value: address(this).balance}(); //wasn't able to do validation with this
}
```

to this

```js
  modifier checkEpoch(bool isOpenForWithdrawal) {
    uint256 remTime = timeLeft();

    if (isOpenForWithdrawal) {
        require(remTime >= 0, 'The deadline has already passed');
    } else {
      require(remTime == 0, 'There is still some time left');
    }
    _;
  }

  function execute() public checkEpoch(false) {
    uint256 contractBalance = address(this).balance;

    //check threshold
    require(contractBalance >= THRESHOLD, 'threshold not reached');

    //? send to external contract
    (bool sent, ) = address(exampleExternalContract)
    .call{value: contractBalance}(abi.encodeWithSignature('complete()'));
    console.log(unicode'\n\t 📤 sending to external contract...\n');
    require(sent, 'unable to send funds to external contract');
  }
```

Started testing for revert error e.g When the threshold isn't reached, calling `execute` shouldn't be possible.

```js
await expect(stakerContract.execute()).to.be.revertedWith(
	"threshold not reached"
);
await expect(stakerContract.execute()).to.be.reverted;
```

Some Errors I faced while working on the challenge.

0. Spent almost three days trying to figure out why the artefacts for the `Staker` contracts weren't being generated. Turns out the latest update to vs-code had turned it off.
1. Came across an overflow error trying to convert the balances using to `toNumber()` method for BigNumber, and instead I should have
2. While trying to deploy the contract using `hard-hat deploy` and NOT the deploy script, I got this error, `nonce has already been used`. It occurred when I chose to increase the gas fees to get my contracts deployed quicker. Choosing `continue waiting` got rid of the error
3. The tests in do no reflect the 72hrs change suggested at the end of the challenge.

```js
const balAfterStaking = await stakerContract.balances(owner.address);
console.log(
	"\n\tbalance after staking",
	ethers.utils.formatEther(balAfterStaking)
);
```

## Next Steps

-   Get started working on challenge 2 and 3.

See you next week for the next one. Should you come across any error or technically incorrect information here, reach out [@jimii_47](https://twitter.com/jimii_47)
