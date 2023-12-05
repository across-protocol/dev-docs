---
description: Use Across to bridge + execute a transaction
---

# Composable Bridging

## **What is Composable Bridging:?**

You can instruct Across to execute a transaction upon filling your deposit on the destination chain. This transaction would be executed atomically with the fill transaction.

**NOTE: The transaction that gets executed on the destination chain must be non-reverting otherwise user deposits may risk getting locked.**

## How does it work?&#x20;

When a relayer fills your deposit by calling `fillRelay()` on the destination SpokePool, if the deposit has a message attached, then the [SpokePool will attempt to call `handleAcrossMessage()`](https://github.com/across-protocol/contracts-v2/blob/6001222d7b7a4fd262ee5503a6169b0664fdb9cd/contracts/SpokePool.sol#L1279) on your recipient address and pass in the following params:&#x20;

* [`handleAcrossMessage(address tokenSent, uint256 amount, bool fillCompleted, address relayer, bytes memory message)`](https://github.com/across-protocol/contracts-v2/blob/6001222d7b7a4fd262ee5503a6169b0664fdb9cd/contracts/SpokePool.sol#L20)

## Requirements&#x20;

* The deposit `message` is not empty
* The `recipient` address is a contract on the destinationChainId that implements a public `handleAcrossMessage(address,uint256,bool,address,bytes)` function, and this function must be non-reverting&#x20;
* The additional gas cost to execute the above function is compensated for in the deposit's [`relayerFeePct`](../how-across-works/overview/fee-model.md#relayer-fees)

## Detailed instructions

* Construct your `message`&#x20;
* Use the Across API to get an estimate of the `relayerFeePct` you should set for your message and recipient combination&#x20;
* Call `deposit()` passing in your message&#x20;
* Once the relayer calls `fillRelay()` on the destination, your recipient's `handleAcrossMessage` will be executed

## Example: Implementing a Bridge and Unwrap

* Imagine that I want to bridge ETH from Ethereum to Optimism and receive ETH, not wrapped WETH on Optimism
* I will deploy the following contract on Optimism which unwraps received WETH into ETH and sends to a designated `owner` EOA

```solidity
contract MyUnwrapper {
    WETHInterface weth;
    addess payable owner;
    constructor() public {
        owner = msg.sender;
    }
    function handleAcrossMessage(
        address tokenSent, 
        uint256 amount, 
        bool, // fillCompleted unused
        address, // relayer is unused 
        bytes memory // message is unused
    ) external { 
        require(tokenSent == address(weth), "received token not WETH");
        weth.withdraw(amount);
        (bool sent, bytes memory data) = owner.call{value: amount}("");
        require(sent, "Failed to send Ether");
    } 
}
```

* Call Across-API’s [/suggested-fees](across-api.md#suggested-fees) endpoint with params `?token=0xWETH-on-ethereum-address&destinationChainId=10&amount= x&originChainId=1&recipient=`MyUnwrapper`.address&message=0x1234`&#x20;
* Here we set `message` to something useless but not-zero so that the destination `SpokePool` ultimately calls `handleAcrossMessage` on `MyUnwrapper`
