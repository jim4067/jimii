+++
title = "Week Two of EthIndia"
date = 2023-02-17
category = "Prog"

[taxonomies]
tags = [ "ethereum"]
+++

Week two of the fellowship and I learned about ERC20 Tokens and How not Random block hashes are given the deterministic nature of Ethereum. 

<!-- more -->

## [challenge two](https://speedrunethereum.com/challenge/token-vendor)

Neat things I took note of.

1. Testing of emitted events can be done like so.

    ```js
    console.log("\t", " 💸 Buying...");
    const buyTokensResult = await vendor.buyTokens({
    	value: ethers.utils.parseEther("0.001"),
    });
    console.log("\t", " 🏷  buyTokens Result: ", buyTokensResult.hash);

    console.log("\t", " ⏳ Waiting for confirmation...");
    const txResult = await buyTokensResult.wait();
    expect(txResult.status).to.equal(1);

    await expect(buyTokensResult).to.emit(vendor, "BuyTokens");
    ```

2. Learnt about the `override` keyword which allows the return type of an inherited function to be changed.
   To allow the function in the parent contract to be overridden, it needs to have the `virtual` keyword.
3. `transfer` moves the tokens from your Token contract while `transferFrom` moves `allowed` tokens that have been `approved` from one address to another.

For further reading check the [open zeppelin](https://docs.openzeppelin.com/contracts/2.x/api/token/erc20) implementation of the ERC20 contract.


## [challenge three](https://speedrunethereum.com/challenge/dice-game)

This challenge was relatively easy to work out as I just needed to copy the functions from the Dice game onto my exploitive contract.

Check out the verified contract [here](https://goerli.etherscan.io/address/0xb9e3f57e4f800f68105a0e34a50369a21d1b0749#code)