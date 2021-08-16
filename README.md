# Tutorial - How to use Chainlink Keepers

Smart contracts are passive autonomous agents, they can't trigger or initiate their own functions at arbitrary times or under arbitrary conditions.

Changes in a smart contract’s state will only occur when a transaction is initiated by another account, like a user, oracle, or other contract.

Chainlink Keepers is a decentralized network of nodes that are incentivized to perform  registered jobs (Upkeeps). This means that they can be used to check conditions and send transactions for smart contracts according to pre-established rules and in this way be the active agent that interacts with a smart contract.

In this tutorial I will show you step-by-step how to create and deploy a smart contract called counter with a rule to be increased and then use Chainlink Keepers to update the counter automatically, according to the rule defined in the contract.

# Overview

Here is a summary of the steps to be taken to experience Chainlink Keepers:

1. Know Chainlink Keepers
2. Create the smart contract counter
3. Understand the smart contract
4. Publish it on Kovan network
5. Verify the smart contract on Etherscan
6. Register a new UpKeep in Keepers App
7. Follow the result

# Requirements

(Needed background and assumptions)

* Have Metamask installed
* Have some Ethers on Kovan network
* Have Link token on Kovan network
* Used Remix before
* Know how to verify a source code in Etherscan

# Making Keeper Compatible Contracts

[https://docs.chain.link/docs/chainlink-keepers/compatible-contracts/](https://docs.chain.link/docs/chainlink-keepers/compatible-contracts/)

## KeeperCompatibleInterface

The smart contract Counter must have these functions to be keeper-compatible:

* checkUpkeep - Checks if the contract requires work to be done.
* performUpkeep - Performs the work on the contract, if instructed by checkUpkeep.

There are some parameters that will not be used and also not be explained here: checkData and performData

You can learn more about it in [compatible-contracts/](https://docs.chain.link/docs/chainlink-keepers/compatible-contracts/)

# Create a smart contract

1. Click on the second button on the left side - file explorer
2. Click on the button create a new file

![Remix - create a new file](/images/image_1.png)

File name: Counter.sol

Copy the smart contract inline below:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity 0.8.6;

interface KeeperCompatibleInterface {
    function checkUpkeep(bytes calldata checkData) external returns (bool upkeepNeeded, bytes memory performData);
    function performUpkeep(bytes calldata performData) external;
}

contract Counter is KeeperCompatibleInterface {

    uint public counter;    // Public counter variable

    // Use an interval in seconds and a timestamp to slow execution of Upkeep
    uint public immutable interval;
    uint public lastTimeStamp;    

    constructor(uint updateInterval) {
      interval = updateInterval;
      lastTimeStamp = block.timestamp;
      counter = 0;
    }

    function checkUpkeep(bytes calldata checkData) external view override returns (bool upkeepNeeded, bytes memory performData) {
        upkeepNeeded = (block.timestamp - lastTimeStamp) > interval;
        performData = checkData;
    }

    function performUpkeep(bytes calldata performData) external override {
        lastTimeStamp = block.timestamp;
        counter = counter + 1;
        performData;
    }
}

```

Paste it into Remix here:

![Remix - Counter.sol](/images/image_2.png)

# Counter.sol

First of all, the interface KeeperCompatibleInterface is declared in the same file.

We normally create separate files, but here it is this way just to make it easier to check the source code from the remix in etherscan.

Take a look in the characteristics of the contract Counter:

It is created by inheriting the interface KeeperCompatibleInterface.

It has this public variables:

* counter to store a number 

* interval in seconds, immutable

* lastTimeStamp to store the last time that the counter was updated

The constructor, which is executed only in the moment of publish the smart contract, initialize the variables:

* counter starts in 0

* interval is defined here and cannot be changed - it is a parameter in the constructor

* lastTimeStamp is the moment the smart contract is created

The function checkUpkeep defines the condition to increase the counter and it will be called by Upkeep

The function performUpkeep increases the counter and updates the timestamp. It is called by Upkeep, when checkUpkeep returns true.

# Compile a smart contract

If you enabled auto-compile, the smart contract is already compiled and a green light will appear next to the third button from the left - Solidity compiler.

![Remix - compilation successful](/images/image_3.png)

If you haven't enabled it, perform the following steps:

1. Click on the 3rd button at the left side, selecting Solidity compiler

2. Click on the button Compile Counter.sol.

3. Check the green signal on the 3rd button with the message compilation successful.

# Publish on Kovan Testnet

## Requirements

1. Selected the Kovan network in Metamask.

2. Have Kovan Ethers in your wallet.

3. In Remix, at Deploy and run transactions, under Environment, make sure you have selected the Injected Web3 option.

In Remix, on the left side, locate the fourth button: Deploy and run transactions.

In the dropdown contract, select Counter

In the right side of the button deploy, fill the parameter updateInterval

I will use 30, for 30 seconds

![Remix - deploy](/images/image_4.png)

Click the button Deploy.

It will open a popup window on Metamask, to confirm the transaction set up by Remix to publish the smart contract.

Click confirm / submit in Metamask.

# Check it on Remix

After the smart contract is published using Remix, we can see your instance in the left panel, at the bottom of deploy and run transactions.

Go to the section deployed contracts, locate the smart contract and expand it.

Click on the blue buttons and confirm that they are the values defined in the constructor:

![Remix - blue buttons](/images/image_5.png)

# Verify the smart contract on Etherscan

In order to use Keepers you need to verify the source code in Etherscan.

Copy the smart contract address from Remix.

Go to [kovan.etherscan.io/](https://kovan.etherscan.io/) and paste the address

For example, this is mine:

[0x8ffbe4e71ff4aafb1e85c78fa1eb00085277016e](https://kovan.etherscan.io/address/0x8ffbe4e71ff4aafb1e85c78fa1eb00085277016e) 

Go to the tab Contract and click on Verify and Publish


Fill this info:

* Compiler Type: Solidity Single file

* Compiler Version: 0.8.6+commit.11564f7e

* Open Source License Type: MIT License (MIT)

* I agree to the terms of service

![Etherscan - screen 1](/images/image_6.png)

Click on Continue button

Copy the source code from Remix and paste it on Etherscan.

![Etherscan - screen 2](/images/image_7.png)


Click on I’m nor a robot and then in the button Verify and Publish.

The result will be like this:

![Etherscan - screen 3](/images/image_8.png)

Take a look on mine after the verification:

[https://kovan.etherscan.io/address/0x8ffbe4e71ff4aafb1e85c78fa1eb00085277016e#code](https://kovan.etherscan.io/address/0x8ffbe4e71ff4aafb1e85c78fa1eb00085277016e#code)

# Get Link Token

Make sure you have some LINK tokens in Kovan.

You need it in order to use in the Keepers App.

You can get it in the faucet

[https://kovan.chain.link/](https://kovan.chain.link/)

# Keepers App

Once you have deployed a Keeper compatible contract, we need to register it with the Chainlink Keeper Network, creating an UpKeep.

After your smart contract is registered, you can use directly the KeeperRegistry functions

[https://etherscan.io/address/0x109A81F1E0A35D4c1D0cae8aCc6597cd54b47Bc6#code](https://etherscan.io/address/0x109A81F1E0A35D4c1D0cae8aCc6597cd54b47Bc6#code)

Let’s go to the Keepers app:

[keepers.chain.link/](https://keepers.chain.link/) 

## Connect your wallet

Connect your wallet with the button in the top right corner.
Select the Kovan network.

![Keepers - connect wallet 1](/images/image_9.png)

![Keepers - connect wallet 2](/images/image_10.png)

## Register new upkeep

Click on the Register new upkeep button

![Keepers - Register new upkeep button](/images/image_11.png)

## Fill out the registration form

The information you provide will be publicly visible on the blockchain. 

Your email address will be encrypted.

For the Counter contract, I suggest you fund it with 20 LINK tokens.

Upkeep name: Counter

Upkeep address: the address of your smart contract

Admin address: it is filled automatically with the connected wallet

Gas limit: 200000

Check data (Hexadecimal): 
Starting balance (LINK): 20

Take a look in mine example:

![Keepers - Register new upkeep](/images/image_12.png)

Click on Register Upkeep button

Confirm the transaction in Metamask

Check it out the confirmation, this is an example

![Keepers - Register new upkeep - confirm](/images/image_13.png)

[0xd8d7bff6b116c0b8b0375ecc8e60864b0cb0f39846b53e7a2c1bd4d1f4d9e2c3](https://kovan.etherscan.io/tx/0xd8d7bff6b116c0b8b0375ecc8e60864b0cb0f39846b53e7a2c1bd4d1f4d9e2c3)

Click on view Upkeep

![Keepers - Register new upkeep - view](/images/image_14.png)

Wait a minute and you will see the last keeper field be updated

![Keepers - Register new upkeep - result](/images/image_15.png)

Check it out the History

![Keepers - Register new upkeep - history](/images/image_16.png)

# Query in Remix

Come back to Remix, and click on the button counter again. It will increase!

Also check the lastTimeStamp, it will be updated.

![Remix - query](/images/image_17.png)

# Final considerations

This was a step-by-step tutorial to show how to create a basic example for Keepers.
Several concepts were not covered in depth, but you can learn directly by reading the official documentation:

[docs/chainlink-keepers/introduction/](https://docs.chain.link/docs/chainlink-keepers/introduction/)

I showed you how to create and deploy a Counter smart contract compatible with Keeper, verify it on Etherscan, use the Keepers app creating an UpKeep and see the Counter be updated automatically!

I hope this tutorial has been helpful and I’d appreciate your feedback. Share it if you like it :)
