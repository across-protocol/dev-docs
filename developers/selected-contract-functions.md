---
description: Explanation of most commonly used smart contract functions
---

# Selected Contract Functions

## SpokePool state-modifying functions

The [spoke pool](https://github.com/across-protocol/contracts-v2/blob/19d3c5a3c62839f530f0db0a6801831890fb0d79/contracts/interfaces/SpokePoolInterface.sol#L90) contract is where deposits are originated and fulfilled.&#x20;

### `deposit`&#x20;

This triggers a deposit request of tokens to another chain with the following parameters. The `originChainId` is automatically set to the chain ID on which the SpokePool is deployed. For example, sending a deposit from the `Optimism_SpokePool` will set its `originChainId` equal to 10.&#x20;

Note on sending ETH: `deposit` is a `payable` function meaning it is possible to send native ETH instead of wrapped ETH (i.e. WETH). If you choose to send ETH, you must set `msg.value` equal to `amount`.

Note on approvals: the caller must approve the SpokePool to transfer `amount` of tokens.

Note on `amount` limits: If the `amount` is set too high, it can take a while for the deposit to be filled depending on available relayer liquidity. If the `amount` is set too low, it can be unprofitable to relay regardless of the relayer fee %. Query the suggested max and min limits [here](across-api.md#querying-limits). The contracts will not revert if the `amount` is set outside of the recommended range, so it is highly recommended to set `amount` within the suggested limits to avoid locking up funds for an unexpected length of time.

Note on setting `quoteTimestamp`:&#x20;

1. Call the read-only function `getCurrentTime()` to get the current UNIX timestamp on the origin chain. e.g. this could return: 1665418548.
2. Call the read-only function `depositQuoteTimeBuffer()` to get the buffer around the current time that the `quoteTimestamp` must be set to. e.g. this could return: 600.
3. `quoteTimestamp` must be <= `currentTime + buffer` and >= `currentTime - buffer`.

| Type    | Name               | Explanation                                                                                                                                                                                                                                                                                                   |
| ------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| address | recipient          | Receiver of bridged funds on destination chain.                                                                                                                                                                                                                                                               |
| address | originToken        | Bridged token address on origin chain.                                                                                                                                                                                                                                                                        |
| uint256 | amount             | Amount of tokens to send on origin chain. Receiver receives this amount minus fees on destination chain.                                                                                                                                                                                                      |
| uint256 | destinationChainId | Where bridged funds should be sent to recipient. Recipient will receive the equivalent of the `originToken` on the destination chain. The mapping of  destination and origin tokens can be queried  [here](across-api.md#finding-available-routes).                                                           |
| uint64  | relayerFeePct      | % of `amount` to pay to relayer. Must be less than 0.5e18 (i.e. 50%). Suggested fees can be queried [here](across-api.md#calculating-suggested-fees). Be careful: if this % is set too low, relayers could be disincentivized to fill this deposit quickly. This can be sped up by calling `speedUpDeposit`.  |
| uint32  | quoteTimestamp     | Timestamp of deposit. Used by relayers to compute the [LP fee %](broken-reference) for the deposit. Must be within`depositQuoteTimeBuffer()` of the current time.                                                                                                                                             |
| bytes   | message            | Data that can be passed to the `recipient` if it is a contract. This is not officially supported by relayers yet, so all deposits should set this to ``"" (i.e. `bytes` of length 0, or the "empty string")`` for now or they risk not being filled. Stay tuned for updates about this feature!               |
| uint256 | maxCount           | This parameter can be set to protect against front-running in the new UBA fee model. Stay tuned for updates about this feature. For now, set this to UINT256.MAX\_UINT to avoid deposit reverts.                                                                                                              |

### `speedUpDeposit`&#x20;

Some of a pending deposit's parameters can be modified by calling this function. If a deposit has been completed already, this function will not revert but it won't be able to be filled anymore with the updated params. It is the responsibility of the depositor to verify that the deposit has not been fully filled before calling this function.

A depositor can request modifications by signing a hash containing the updated details and information uniquely identifying the deposit to relay. This information ensures that this signature cannot be re-used for other deposits.

We use the [EIP-712](https://eips.ethereum.org/EIPS/eip-712) standard for hashing and signing typed data. Specifically, we use the [version of the encoding known as "v4"](https://docs.metamask.io/guide/signing-data.html), as implemented by the JSON RPC method `eth_signedTypedDataV4` in MetaMask.&#x20;

You can see how the message to be signed is reconstructed in Solidity [here](https://github.com/across-protocol/contracts-v2/blob/19d3c5a3c62839f530f0db0a6801831890fb0d79/contracts/SpokePool.sol#L1063).

Successfully calling this function will emit an event `RequestedSpeedUpDeposit` which can be used by relayers to fill the original deposit with the new parameters.  Depositors should assume that the parameters emitted with the highest `relayerFeePct` will be used, since they are incentivized to use the highest fee possible.

Any relayer can use updated deposit parameters by calling `fillRelayWithUpdatedDeposit` instead of `fillRelay.`

| Type    | Name                 | Description                                                                                                                                                            |
| ------- | -------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| address | depositor            | Sender of deposit to be sped up. Does not need to equal `msg.sender`                                                                                                   |
| int64   | updatedRelayerFeePct | New relayer fee % that relayers can use when calling `fillDepositWithUpdatedDeposit`                                                                                   |
| uint32  | depositId            | UUID of deposit to be sped up                                                                                                                                          |
| address | updatedRecipient     | New recipient of deposit.                                                                                                                                              |
| bytes   | updatedMessage       | Updated data that is sent to `updatedRecipient`. As described in section above, this should be set to 0x for the forseeable future.                                    |
| bytes   | depositorSignature   | Signed message containing contents [here](https://github.com/across-protocol/contracts-v2/blob/19d3c5a3c62839f530f0db0a6801831890fb0d79/contracts/SpokePool.sol#L1029) |
