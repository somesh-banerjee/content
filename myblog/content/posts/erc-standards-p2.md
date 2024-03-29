+++
title = "Erc Standards Part 2"
date = "2023-01-12T20:08:49+05:30"
author = "Somesh Banerjee"
authorTwitter = "banerjee_somesh"
cover = ""
tags = ["Solidity", "Smart-Contract", "ERC Token Standards"]
keywords = ["Solidity", "Ethereum", "Smart-Contract", "ERC", "ERC Token Standards"]
description = ""
showFullContent = false
readingTime = true
hideComments = false
+++

In my last [post](../erc-standards) on ERC standards, I discussed the basic ERC standards like ERC20, ERC721, ERC1155, etc. In this post, I will go over some more ERC standards finalized by the community.

# ERC 1363: Payable Token​

Regarding ERC20 tokens, we generally use them for payment purposes like paying for an NFT, a service, etc. But after we transfer the ERC20 tokens, somebody on the other side has to initiate the next transaction. This results in paying the gas price twice for a single process(by process, I mean a set of steps like paying and getting an NFT). The payable token is introduced to solve this issue, where the ERC 1363 contract initiates the next transaction after payment is made.

# ERC 2981: NFT Royalty Standard​

This ERC is developed to implement royalty features in the marketplaces. It is an extension for ERC721 and ERC1155 to give royalties to the creators. The standard introduces the following function to return royalty details like how much to transfer and whom to transfer.
```js
function royaltyInfo(
    uint256 _tokenId,
    uint256 _salePrice
) external view returns (
    address receiver,
    uint256 royaltyAmount
);
```

If we use this standard, royalty is not guaranteed as the ERC only gives royalty information. Implementing it relies on the marketplace where the transfer is happening.

# ERC 3525: Semi-Fungible Token​

Semi Fungible token is similar to ERC 721 tokens, where each type of token has a unique token Id, but two different token Ids are considered fungible if they share the same slot. This standard identifies the tokens by <ID, SLOT, VALUE>. The VALUE property tells the total supply of the token ID. We can transfer tokens between different IDs if they have the same slot.

# ERC 4519: Non-Fungible Tokens Tied to Physical Assets​

This token standard is built for smart physical devices with an address in the network. The ERC 721 only tracks ownership records and not usage rights. EIP 4519 was proposed to solve this issue, where the device, the token, and the user interact with each other to track all the records. To understand in-depth, read the [paper](https://eips.ethereum.org/assets/eip-4519/sensors-21-03119.pdf) published in Sensors journal.

# ERC 4907: Rental NFT​

This standard is an extension of ERC 721. Here each NFT has an owner and a user. These two properties can be two different wallet addresses. The user will have limited control over the NFT, like he can't transfer the NFT. The owner can set whom to make the user and for what duration, like rent.

# ERC 5023: Shareable NFT​

Shareable NFTs can be implemented in both ERC 721 and ERC 1155. In this standard, an NFT is shared with another user/wallet by reminting the same token and sending the newly minted token to the new owner. The freshly minted token represents the same digital asset but has a unique ID. The original token remains with the old owner.

# ERC 5192: Minimal soul bound token​

This standard is an extension of ERC721. SoulBound token, as you can understand from the name, is a token bound to a single soul or wallet. To implement soulbond tokens,  the transfer function of the ERC721 token is disabled.

# ERC 5484: Consensual Soul bound Tokens​

This standard is an extension of ERC721 and is almost similar to ERC 5192 but with one addition, i.e., the property of burn authorization. Burn Authorization means who can burn the token. It can be only the issuer, only the receiver, or both of them, or neither of them.

# ERC 5528: Refundable Fungible Token​

This standard is an extension of ERC 20 tokens. When we send ERC 20 tokens for a task, and that task fails or is not completed, we don't get our tokens back automatically. In ERC 5528, an escrow contract is used to solve this issue. Let's understand this with a token swap example. The seller creates an escrow contract with a given condition.​ Both buyer and seller send the token to the escrow contract. If the requirement is fulfilled, then tokens are transferred to the respective receiver.​ If it fails, the buyer can withdraw the tokens submitted by him.​


**References:**
1. [ERC | Ethereum Improvement Proposals](https://eips.ethereum.org/erc)