# Migration from V2 to V3

## Introduction

Across V3 is coming soon. This guide is intended to help developers who integrate with Across to prepare their codebases for the upgrade. Please note that this page is a work in progress, and more sections will be added to help with different types of integrations.

## API migration guide

### Overview

Across v3 redesigns how fees are handled when creating deposits. In a nutshell, the calculation of fees will be simplified and replaced by`inputAmount`/`outputAmount` arguments. This will impact the response data of the API and also how to call the `deposit` function of a `SpokePool` contract. Note that these changes **won't be breaking short-term but are actionable mid- to long-term.**

### Non-breaking changes

For a seamless upgrade, there will be no breaking changes for the existing v2 interfaces.

#### \[Updated] `relayFeePct` will now include `lpFeePct`

In Across v2, the request `GET /suggested-fees` returned

```ts
type FeesResponse = {
    // ... other fields
    capitalFeePct: string;
    capitalFeeTotal: string;
    relayGasFeePct: string;
    relayGasFeeTotal: string;
    relayFeePct: string; // capitalFeePct + gasFeePct
    relayFeeTotal: string; // capitalFeeTotal + gasFeeTotal
    lpFeePct: string;
}
```

It was then expected to call the method `deposit` of a `SpokePool` like

```ts
const tx = await spokePool.deposit(
    // ... other args
    feesResponse.relayFeePct // capitalFeePct + gasFeePct
)
```

The `lpFeePct` was not required as an argument and automatically derived based on the `quoteTimestamp`. But if you wanted to show the total bridge fee to the user, then you would have to sum them up like

```ts
const totalBridgeFeePct =  relayFeePct + lpFeePct
```

In Across v3, we need to pass the `lpFeePct` as part of the total fee when calling the `deposit` function. In order to be backwards-compatible, the API now returns for `GET /suggested-fees`

```diff
type FeesResponse = {
    // ... other fields
    capitalFeePct: string;
    capitalFeeTotal: string;
    relayGasFeePct: string;
    relayGasFeeTotal: string;
-   relayFeePct: string; // capitalFeePct + gasFeePct
+   relayFeePct: string; // capitalFeePct + gasFeePct + lpFeePct
-   relayFeeTotal: string; // capitalFeeTotal + gasFeeTotal
+   relayFeeTotal: string; // capitalFeeTotal + gasFeeTotal + lpFeeTotal
-   lpFeePct: string;
+   lpFeePct: "0";
}
```

There are no changes to the interface and therefore no changes are required for how you call the `deposit` function or calculate the total bridge fee. If you want to show the correct fee breakdown though, some changes are needed (see here).

#### \[Added] New v3 `fees` structure

If you want to show the correct detailed fee breakdown, you can use the newly added `v3` properties of the `GET /suggested-fees` response data

```diff
type FeesResponse = {
     // ... other fields
+    totalRelayFee: { // relayerCapitalFee + relayerGasFee + lpFee
+        pct: string;
+        total: string;
+    };
+    relayerCapitalFee: {
+        pct: string;
+        total: string;
+    };
+    relayerGasFee: {
+        pct: string;
+        total: string;
+    };
+    lpFee: {
+        pct: string;
+        total: string;
+    };
}
```

Using the values of the new fees struct, you need to call the `deposit` function like

```ts
const tx = await spokePool.deposit(
    // ... other args
    feesResponse.totalRelayFee.pct
)
```

### Breaking changes (mid- to long-term)

Even though there are no actionable changes short-term, some change will be actionable mid- to long-term.

#### \[Deprecated] `deposit` function will be replaced by `depositV3`

The v2 `deposit` function will be deprecated and replaced by the `depositV3` function. This new function has a different signature and does not expect a `relayFeePct`. Instead it requires an `outputAmount`

```solidity
function depositV3(
    address depositor,
    address recipient,
    address inputToken,
    address outputToken,
    uint256 inputAmount,
    uint256 outputAmount, // <-- replaces fees
    uint256 destinationChainId,
    address exclusiveRelayer,
    uint32 quoteTimestamp,
    uint32 fillDeadline,
    uint32 exclusivityDeadline,
    bytes calldata message
) public payable
```

In order to set the correct `outputAmount`, the caller needs to send a request `GET /suggested-fees` and subtract the returned fees from the `inputAmount` like

```ts
// Assuming inputToken equals outputToken
const outputAmount = inputAmount - feesResponse.totalRelayFee.total
const tx = await spokePool.depositV3(
    // ... other args
    outputAmount
)
```

#### \[Deprecated] Flattened fees `*Pct`/`*Total` fields will be removed

As described here, the v3 fees have a different structure now. Eventually the redundant fields `capitalFeePct`, `capitalFeeTotal`, `relayGasFeePct`, `relayGasFeeTotal` and `lpFeePct` will be removed

```diff
type FeesResponse = {
    // ... other fields
-    capitalFeePct: string;
-    capitalFeeTotal: string;
-    relayGasFeePct: string;
-    relayGasFeeTotal: string;
-    relayFeePct: string; // capitalFeePct + gasFeePct
-    relayFeeTotal: string; // capitalFeeTotal + gasFeeTotal
-    lpFeePct: string;
+    totalRelayFee: { // relayerCapitalFee + relayerGasFee + lpFee
+        pct: string;
+        total: string;
+    };
+    relayerCapitalFee: {
+        pct: string;
+        total: string;
+    };
+    relayerGasFee: {
+        pct: string;
+        total: string;
+    };
+    lpFee: {
+        pct: string;
+        total: string;
+    };
}
```
