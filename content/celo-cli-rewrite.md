+++
title = "Rewriting the CELO CLI in Rust"
date = 2023-01-10
category = "Prog"
draft = true

[taxonomies]
tags = ["rust", "cli"]
+++

I'll start by quoting a tweet by
[Georgios Konstantopoulos]() Paradigm CTO. The rate at which you develop is always bound by how good your tools are. [tweet](https://twitter.com/gakonst/status/1604524993941209089?s=20)

Some of the reasons that prompted me to do a rewrite of the cli

1. Installation - To install the celocli, you need to be running a specific version of node(node). This can be achieved by using nvm but when I change to work with another project with a different node version I now cannot access celocli which I found to be frustrating.

2. It could be improved even further to use subcommands e.g `celocli config:get` could be simplified to `celocli config get` which is more common in cli tool and also should enable more composability.

3. While working in the Solana ecosystem, anchor was really nice to work with. It has so many features like setting up your dev env, deploying and verifying contracts. I think that celocli should also encompass all these features and add more. Some tools that make dev work easier including `hardhat` and `foundry` do not come with default celo configs as I come to know.

4. Allow the users to create numerous accounts and have them saved in his environment, give him the option to name and rename accounts and let them show up when `account list` flag is passed. Currently they do not.

## Celo compose

-   There is some wait time when initializing a new project.


## How I want it to look
- let it be plugin based. Due to size and bandwidth constraints, we ship a the cli that has the a plugin feature just like cargo. 
For example, if a user only wants to use the cli to  create celo projects, he/she can only install the cli with that plugin installed. If later down the road he wants to use the vote functionality he can download the vote plugin and this will enable him to vote on things. If he needs the account functionality he can also install it.

for now I am thinking that the plugins will consist of the following choices
        1. account - to create `0x` based accounts and use them for deploying contracts, etc
        2. vote - enable user to vote on things(don't know how this works)
        3. composer - help devs scaffold their projects faster
        4. node and validator(what is the difference)


- give the user to install the cli with all the plugins, zero plugins or a select few of the plugins that he will need.

- for the composer to reduce the size, we can have it cache the files in the .local directory for *offline* usage and update it when the origin changes. (TDB)
        - In other words, we want it to be an offline first tool 

- create a celo version manager

## Improvements
- improve the documentation giving examples