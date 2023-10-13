# Disputing Root Bundles

### About

Across requires proposals and disputes to be accompanied by a bond. This bond is returned if the proposal or dispute is correct, and is sacrificed if it is found to be incorrect. This protects against attempts to incorrectly move funds, as well as spam and other denial of service attempts.

The [Across Bond Token](https://github.com/across-protocol/contracts-v2/blob/f5414207c164a116efafd0a0adf6e440aa88f85f/contracts/BondToken.sol) ([ABT](https://etherscan.io/token/0xee1DC6BCF1Ee967a350e9aC6CaaAA236109002ea)) is the bond collateral required by the [HubPool](https://etherscan.io/address/0xc186fa914353c44b2e33ebe05f21846f1048beda) contract. This is a WETH-like contract that is minted in return for depositing Ether, and can be redeemed for the underlying Ether at any time. [ABT](https://etherscan.io/token/0xee1DC6BCF1Ee967a350e9aC6CaaAA236109002ea) implements custom ERC20 [transferFrom()](https://github.com/across-protocol/contracts-v2/blob/f5414207c164a116efafd0a0adf6e440aa88f85f/contracts/BondToken.sol#L73) logic in order to limit the addresses that are able to make [HubPool](https://etherscan.io/address/0xc186fa914353c44b2e33ebe05f21846f1048beda) root bundle proposals.

### Manual Dispute Procedure

1. Check the required bond token and amount (nominally 0.45 ABT) by calling [bondToken()](https://etherscan.io/address/0xc186fa914353c44b2e33ebe05f21846f1048beda#readContract#F2) and [bondAmount()](https://etherscan.io/address/0xc186fa914353c44b2e33ebe05f21846f1048beda#readContract#F1) on the [HubPool](https://etherscan.io/address/0xc186fa914353c44b2e33ebe05f21846f1048beda).
2. Mint the bond token as necessary by caling [deposit()](https://etherscan.io/address/0xee1DC6BCF1Ee967a350e9aC6CaaAA236109002ea#writeContract#F2) on the BondToken contract.
3. Ensure that the [HubPool](https://etherscan.io/address/0xc186fa914353c44b2e33ebe05f21846f1048beda) has permission to pull the bond during the dispute. Increase the allowance as necessary by calling [appprove()](https://etherscan.io/address/0xee1dc6bcf1ee967a350e9ac6caaaa236109002ea#writeContract#F1) on the BondToken contract. The address to approve is [0xc186fa914353c44b2e33ebe05f21846f1048beda](https://etherscan.io/address/0xc186fa914353c44b2e33ebe05f21846f1048beda).&#x20;
4. Call [disputeRootBundle()](https://etherscan.io/address/0xc186fa914353c44b2e33ebe05f21846f1048beda#writeContract#F4) on the [HubPool](https://etherscan.io/address/0xc186fa914353c44b2e33ebe05f21846f1048beda).

### Automated Dispute Procedure

The Across [relayer-v2](https://github.com/across-protocol/relayer-v2) repository contains a utility script that automates each of the above steps. Prerequisites are:

1. The relayer-v2 package must be installed.
2. The mnemonic for an EOA must be set in the relayer-v2 `.env` file.
   1. The configured EOA must be funded with at least 0.45 ABT or ETH  (1 ABT == 1 ETH), **plus** additional ETH for gas to handle the necessary deposit, approval and/or dispute transactions.
   2. It is sufficient for the entire amount to be held in ETH, since the dispute script automates the steps of minting ABT and approving the HubPool to spend it.
   3. The actual amounts are subject to change based on the prevailing gas price at the time of the dispute, and the configured bond amount.

#### Installation

```
$ git clone https://github.com/across-protocol/relayer-v2.git
$ cd relayer-v2
$ yarn install && yarn build

# Copy the predefined sample config and update the MNEMONIC variable in
# .env to match the relevant mnemonic.
$ cp .env.example .env
```

#### Execution

```
$ yarn dispute

# The dispute script will dump information about the Bond Token and 
# latest HubPool proposal. If necessary, it will automatically mint the 
# requisite amount of the bond token and will approve the HubPool to use 
# it. At the conclusion, the script will provide the transaction hash of
# the most recent proposal and will request to re-run with the flag
#
#     --txnHash <proposal-transaction-hash>
# 
# Re-running the script with this additional argument will automatically 
# submit a dispute.

$ yarn dispute --txnHash <proposal-transaction-hash>
```
