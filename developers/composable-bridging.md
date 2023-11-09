---
description: Using Across to build a Bridge-and-Swap smart contract
---

# Composable Bridging

## **What is Composable Bridging:?**

You can instruct Across to execute a transaction upon filling your deposit on the destination chain. This transaction would be executed atomically with the fill transaction.

## How does it work?&#x20;

When a relayer fills your deposit by calling `fillRelay()` on the destination SpokePool, if the deposit has a message attached, then the [SpokePool will attempt to call `handleAcrossMessage()`](https://github.com/across-protocol/contracts-v2/blob/6001222d7b7a4fd262ee5503a6169b0664fdb9cd/contracts/SpokePool.sol#L1279) on your recipient address and pass in the following params:&#x20;

* [`handleAcrossMessage(address tokenSent, uint256 amount, bool fillCompleted, address relayer, bytes memory message)`](https://github.com/across-protocol/contracts-v2/blob/6001222d7b7a4fd262ee5503a6169b0664fdb9cd/contracts/SpokePool.sol#L20)

## Requirements&#x20;

* The deposit `message` is not empty&#x20;
* The `recipient` address is a contract on the destinationChainId that implements a public `handleAcrossMessage(address,uint256,bool,address,bytes)` function&#x20;
* The additional gas cost to execute the above function is compensated for in the deposit's [`relayerFeePct`](../how-across-works/overview/fee-model.md#relayer-fees)

## Detailed instructions

* Construct your `message`&#x20;
* Use the Across API to get an estimate of the `relayerFeePct` you should set for your message and recipient combination&#x20;
* Call `deposit()` passing in your message&#x20;
* Once the relayer calls `fillRelay()` on the destination, your recipient's `handleAcrossMessage` will be executed

## Example: Implementing a Bridge and Swap on Optimism&#x20;

* Let’s say we want to call `swap(address inputToken, address outputToken, uint256 inputAmount, uint256 minOutputAmount)` on a DEX on Optimism.&#x20;
* We will be bridging 1,000 USDC from Polygon to Optimism for at least 0.1 WETH.&#x20;
* Let’s implement a contract that handles the swap for us and deploy it on Optimism:&#x20;

```solidity
contract MySwapper { 
    address owner;
    DEXInterface dex = 0xSomeDexAddress; 
    address tokenToReceiveFromSwap = 0xWETHon-optimism-address;
    function handleAcrossMessage(
        address tokenSent, 
        uint256 amount, 
        bool fillCompleted, 
        address, // relayer unused 
        bytes memory message
    ) external { 
        require(fillCompleted,"cannot swap until full fill amount is received"); 
        (address outputToken, uint256 minOutAmount) = abi.decode(message, (address,uint256)); 
        uint256 amountOut = dex.swap(tokenSent, outputToken, amount, minOutAmount);
        IERC20(outputToken).transfer(owner, amountOut);
    } 
}
```

* Compute the `message` we'll pass into `SpokePool.deposit()` on Polygon along with the 1,000 USDC



```typescript
const swapMessage: string = ethers.abi.ABICoder.encode(["address","uint256"], [
"0xWETH-on-optimism", // address outputToken e.g. WETH
"100000000000000000" // uint256 minAmountOut e.g. 0.1e18 WETH
]);
```

* Call Across-API’s [/suggested-fees](across-api.md#suggested-fees) endpoint with params `?token=0xUSDC-on-polygon-address&destinationChainId=10&amount= 1000000000&originChainId=137&recipient=MySwapper.address&message=swapMessage`&#x20;
* This returns a `relayerFeePct` we should set when calling deposit with message=swapMessage, recipient=MySwapper.address , etc.&#x20;
* Once the deposit is filled by a relayer on Optimism, `MySwapper.handleAcrossMessage` will be executed within the fill transaction



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
