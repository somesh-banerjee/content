---
title: "Erc Standards"
date: 2022-05-30T15:55:13+05:30
author: "Somesh Banerjee"
authorTwitter: "banerjee_somesh"
cover: ""
tags: ["Solidity", "Smart-Contract", "ERC Token Standards"]
keywords: ["Solidity", "Ethereum", "Smart-Contract", "ERC", "ERC Token Standards"]
description: ""
showFullContent: false
readingTime: true
draft: false
---

# ERC Standards

ERC stands for Ethereum Request for Comments. This documents contains rules for different token standards. If you want to create any Ethereum based token you have to follow this rules.

# Different ERC Standards

![](/blogs/erc-s/1.jpg)
Image Source: [Blockchain Council](https://www.blockchain-council.org/wp-content/uploads/2022/02/ERC-TOKEN.jpg)

## ERC20
This is is the most common Token standard and is commonly used by ICOs. It is a fungible token where all the token holds the same value. This standard can be used as a currency. The basic rule for this standard are
1. The tokens can be transferred from one account to another
2. The balance for each account can be checked.
3. The Total supply of the token can be checked.
4. The owner can approve a third party to spent some amount of his token 

## ERC721
This is the most basic standard for NFTs(Non Fungible Tokens). NFTs are unique and each token holds a different value. The smart contract keep track of the tokens with the help of Token ID. The token IDs are mapped to owner's wallet address. Owner means the owner of that particular token. 

Most digital assets like images, videos, 3D models are sold in the Metaverse marketplace as this NFT. The token cannot store the digital asset onchain as it will be pretty large. Instead it is stored somewhere else like IPFS or centralized server and the link to the asset is stored in the token as URI. The token mainly represents the ownership of that asset.

You can find my ERC721 Marketplace [here](https://metaverse-market.vercel.app/).

## ERC1155
This token standard is normally used in games where there are different kind of tokens with different quantity. Let say we have a have a game of collecting cards with two two types of cards as token: Common Cards and Rare Cards. The availability of the Common card is 100 and Rare is 20. If we treat each type of cards as ERC20 token then you can say that the ERC1155 is maintaining multiple ERC20 tokens.

Also let say in ERC1155 there are many tokens and each of their quantity / availability is 1, then we can say that the contract is similar to ERC721 where each token is unique.

In ERC1155, the smart contract keep tracks of token with the help of both Token ID and Account Address. It stores the number as how many accounts have the given token Id and how much each account has.

## ERC725
This standard is used for creating digital identity. The identity contract can identify other contracts. This is required because in decentralized governance we need the identity of different users without using any centralized server. 

# Implementing the Token Standards
The easiest way to implement ERC20, ERC721 and ERC1155 token is using [OpenZeppelin Contracts](https://www.openzeppelin.com/contracts). Here the basic functionalities of all the standards are already written and tested. Your token will only inherit those contracts. You can also add your own functions as per the requirement. To start you go the wizard, and then select which standard you want to create. You can select what features you want in your contract like mintable, pausable, burnable, etc and it will be added. Once satisfied you can copy the code to remix and deploy it the network you want to.

# What are EIPs
Ethereum Improvement Proposals (EIPs) describes the standard of Ethereum platform. It says what steps need to follow to make a improvement in the token standards. While creating a new ERC standard the author has to follow these steps.

**References:**
1. [101 Blockchains](https://101blockchains.com/erc-standards/)
2. [Blockchain Council](https://www.blockchain-council.org/ethereum/erc-token-standards/)
3. [OpenZeppelin](https://docs.openzeppelin.com/contracts/4.x/)
4. [EIP-1](https://eips.ethereum.org/EIPS/eip-1)