---
description: >-
  Instructions and examples for calling the smart contract functions that would
  allow third party projects to transfer assets across EVM networks.
---

# Integrating Across into your application

Across was designed as a platform on which third party projects enabling cross chain asset transfer could be built. We are very excited to build together and we've put together this guide which _should_ contain everything you need to integrate with Across.&#x20;

If you have further questions or suggestions for this guide, please send a message to the #developer-questions channel in the [Across Discord](broken-reference).

## How to initiate a deposit

Deposits are initiated from contracts called "SpokePools" deployed on any supported EVM. For example, on Ethereum the contract is named "Ethereum_SpokePool.sol"_, and on Optimism the contract is named "Optimism\_SpokePool.sol".&#x20;

Spoke pool addresses can be found [here](contract-addresses/).

Deposits are triggered via the SpokePool contract's `deposit` function, whose parameters are explained in detail [here](selected-contract-functions.md#deposit). This documentation also explains how to populate parameters like `relayerFeePct` and `quoteTime.`

[Here](https://optimistic.etherscan.io/tx/0xef388f8ace4b07f99fc2826f5830197b91c031aeab7802f3a2d7ab3d25a66dae) is a deposit sent on Optimism. [Here](https://etherscan.io/tx/0xd192b1063f29a5f33780bc0b5997711307046d1421a6010a53fe0a53b7d6ce51) is a deposit on Ethereum

## How to track a deposit

Deposit and corresponding fill events are conveniently scraped by a database and displayed [here](https://github.com/across-protocol/scraper-api).

The database implementation can be found in this [repository](https://github.com/across-protocol/scraper-api).

## How to speed up a deposit that is taking a long time to be picked up by a relayer

The lower a deposit's relayer fee %, the less relayers are incentivized to fulfill it. While a deposit is not fully filled, its relayer fee % can be increased by calling [`speedUpDeposit`](selected-contract-functions.md#speedupdeposit).

## Using the API versus SDK to construct deposit parameters

We [recommend](across-sdk.md#if-i-want-to-integrate-across-into-my-dapp-should-i-use-the-sdk-or-the-api) using the API for the easiest way to query suggested deposit params.

## Commonly asked questions

* Can I use the [Across SDK](across-sdk.md) to submit contract transactions?
  * Not yet, working on it!
