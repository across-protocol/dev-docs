---
description: >-
  Below is an overview of how a bridge transfer on Across works from start to
  finish.
---

# Overview

A user that would like to move funds from chain A to chain B deposits funds into a _Spoke Pool_ on chain A with instructions about where they would like their funds to wind up and the fee that they are willing to pay.

Relayers view these deposits and, once they have verified that the details of the deposit are correct, immediately provide funds to the user on chain B. The user is now satisfied, having received their funds minus any fees and no longer needs to interact with Across.

After the relayer has performed the relay, a proof of that relay and the validity of the original deposit is submitted to the optimistic oracle (OO) and the relayer is reimbursed once this information has been verified by the OO.

The relayer's reimbursement is taken out of a single liquidity pool on Ethereum Mainnet escrowed in a contract called the _Hub Pool_. Liquidity providers ("LP's") to this pool also earn a fee per transfer that is assessed on the user's deposited amount.

The rules for how funds are moved between the L2 _Spoke Pools_ and the L1 _Hub Pool_ to reimburse relayers are explained in [UMIP-157](https://github.com/UMAprotocol/UMIPs/blob/master/UMIPs/umip-157.md). Anyone who wants to move funds between the pools must submit a valid proof to the OO that abides by the rules explained in the UMIP.

To see how this all comes together, check out the chart below showing a complete end-to-end flow of the process.

<figure><img src=".gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

### Resources to learn more

The smart contract code can be found [here](https://github.com/across-protocol/contracts-v2), including implementations of the HubPool and SpokePool.

The smart contracts were [audited by OpenZeppelin](https://blog.openzeppelin.com/uma-across-v2-audit/). The audit report contains a high-level summary of how the smart contract architecture works.

Moreover, [here is a 60-minute explainer ](https://www.youtube.com/watch?v=iuxf6Crv8MI)video of the smart contract architecture. [Slides](https://docs.google.com/presentation/d/1aEq7t7yUfUAFTWug9PLdSDLdZCUJO4q7pGAoB24piZI/edit?usp=sharing) for the explainer video can be found here.
