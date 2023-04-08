+++
title = "Week Three and Four of EthIndia"
date = 2023-03-07
category = "Prog"

[taxonomies]
tags = [ "EthIndia"]
+++

Third and fourth week participating in EthIndia fellowship.

<!-- more -->

## [challenge four](https://speedrunethereum.com/challenge/minimum-viable-exchange)

This challenge involved building a decentralized exchange. This challenge enabled me build an understanding of how DEXs like [Uniswap](https://uniswap.org/) work.

[Version 2](https://uniswap.org/blog/uniswap-v2) of Uniswap relied on the `x * y = k` formula which distributes the user's liquidity across the range of `0 ~ ∞`. This type of AMM is referred to as Constant Product Market Marker. Here is a great [tutorial](https://www.youtube.com/watch?v=JSZbvmyi_LE&list=PLO5VPQH6OWdX-Rh7RonjZhOd9pb9zOnHW&index=45&t=1141s) on how to build one by the smart contract programmer.

Version 3 of Uniswap introduces the concept of [Concentrated Liquidity](https://docs.uniswap.org/concepts/protocol/concentrated-liquidity) which enables LPs to define the the ranges within which they want to provide liquidity. 

I am still trying to wrap my head around the Math used to calculate liquidity but the [whitepaper](https://uniswap.org/whitepaper-v3.pdf) does a fantastic job of explaining it. Worth considering if you would love to know how it works would be the [introductory blog](https://uniswap.org/blog/uniswap-v3).


| ![uniswap curves](/eif3/cpmm-vs-ccpmm.jpeg) | 
|:--:| 
| curve differentiating the capital efficiency between Uniswap v-2 and v-3 |

As shown above and described by Uniswap, LPs can provide liquidity with up to 4000x capital efficiency relative to Uniswap v2, earning higher returns on their capital.

## [challenge nine](https://speedrunethereum.com/challenge/state-channels)

The main purpose of this challenge was to learn about scalability of the Ethereum network.

Blockchains can have security, scalability and decentralization but never three at the same time. This is known as the [blockchain trilemma](https://academy.binance.com/en/articles/what-is-the-blockchain-trilemma). 

Ethereum faces a scalability crisis. An increase of users of users leads to the slowdown of the network and non-viable high transaction costs. There is need for scalability as described by the Ethereum foundation to increase the tx throughput and time taken for transactions to reach [finality](https://academy.binance.com/en/glossary/finality).

Many solutions have been proposed to solve this issue which can be divided into two broad categories, on-chain scaling using [sharding](https://www.web3.university/article/ethereum-sharding-an-introduction-to-blockchain-sharding) or off-chain scaling through [layer-2s](https://ethereum.org/en/layer-2/), [side-chains](https://ethereum.org/en/developers/docs/scaling/sidechains/), roll-ups - [optimistic roll-ups](https://ethereum.org/en/developers/docs/scaling/optimistic-rollups/) and [zk-roll-ups](https://ethereum.org/en/developers/docs/scaling/zk-rollups/), [state channels](https://ethereum.org/en/developers/docs/scaling/state-channels/), [plasma chains](https://ethereum.org/en/developers/docs/scaling/)...etc

In this channels we learned how to use state channels to support frequent transactions.

## [challenge seven](https://speedrunethereum.com/)

During this challenge I worked on a multisig wallet.

This was very useful especially coming from doing the tutorial on building state channels where the two are interwoven together.

## [1inch presentation](https://docs.google.com/presentation/d/1l-qjKWVeTaM3EnRhmO_1oz205LyOVYsbWChsMtJzW3E/edit#slide=id.gc6f80d1ff_0_5)

Worked on a presentation for the DEFI protocol (1inch)[https://1inch.io/]