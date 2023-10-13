# Across API

### Source code:

The API is designed to be run serverlessly (without storing state) and is a wrapper on top of the [SDK](across-sdk.md). Implementation [here](https://github.com/across-protocol/frontend-v2/tree/9011e8e231c59dbaec9d2fbe6a35834b7f8c5ab7).

### Caching & Liveness

Users of the Across API are requested to cache results for no longer than 300 seconds.

The Across API serves data that is derived from the on-chain state of the Across contracts and relayer bots. The on-chain state is subject to change each block, and cached data can quickly become invalid as a result.

### Calculating Suggested Fees

The API uses the Across SDK under the hood, but offers a convenient way to get suggested fees when placing a Deposit transaction.

**Example:**

You can visit this example in your browser: [Link](https://across.to/api/suggested-fees?token=0x7f5c764cbc14f9669b88837ca1490cca17c31607\&destinationChainId=42161\&amount=100000000000).

Or curl it on the CLI:

`curl "https://across.to/api/suggested-fees?token=0x7f5c764cbc14f9669b88837ca1490cca17c31607&destinationChainId=42161&amount=100000000000"`

**Note:** When filling relays, it is strongly recommended to use the Across SDK [relayFeeCalculator](https://github.com/across-protocol/sdk-v2/tree/master/src/relayFeeCalculator). Using the `suggested-fees` API endpoint is done at the relayer's own risk.

#### API Definition

All API calls use `https://across.to/api`as the host.

#### suggested-fees

Path: `/suggested-fees`

Method: `GET`

Query Params

<table><thead><tr><th>Parameter Name</th><th width="260.3333333333333">Description</th><th>Example</th></tr></thead><tbody><tr><td>token</td><td>Address of token contract to transfer. For ETH (or other native tokens, like matic) use, use the wrapped address, like WETH.<br><br>Note: the address provided can be the token address on any chain. In the unlikely event where two different tokens have the same address on different chains, you can use the optional chainId parameter defined below to indicate which chain should be used.</td><td>0x7f5c764cbc14f9669b88837ca1490cca17c31607</td></tr><tr><td>destinationChainId</td><td>The intended destination of the transfer.</td><td>42161</td></tr><tr><td>amount</td><td>Amount of the token to transfer. Note: this amount is in the native decimals of the token. So, for WETH, this would be the amount of human-readable WETH multiplied by <code>1e18</code>. For USDC, you would multiply the number of human-readable USDC by <code>1e6</code>.</td><td>100000000000</td></tr><tr><td>timestamp (optional)</td><td>The quote timestamp used to compute the LP fees. When bridging with across, the user only specifies the quote timestamp in their transaction. The relayer then determines the utilization at that timestamp to determine the user's fee. This timestamp must be close (within 10 minutes or so) to the current time on the chain where the user is depositing funds and it should be &#x3C;= the current block timestamp on mainnet. This allows the user to know exactly what LP fee they will pay before sending the transaction.<br><br>If this value isn't provided in the request, the API will assume the latest block timestamp on mainnet.</td><td>1653547649</td></tr><tr><td>originChainId (optional)</td><td>Used to specify which chain where the specified token address exists. Note: this is only needed to disambiguate when there are matching addresses on different chains. Otherwise, this can be inferred by the API.</td><td>10</td></tr></tbody></table>

Returns a JSON object with the following properties:

| Property Name | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | Example           |
| ------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- |
| relayFeePct   | <p>The percentage of the transfer amount that should go to the relayer as a fee. This is the strongly recommended minimum value to ensure a relayer will perform the transfer under the current network conditions.</p><p></p><p>The value returned in this field is guaranteed to be at least 0.03% in order to meet minimum relayer fee requirements.</p><p></p><p>Note: 1% is represented as <code>1e16</code>, 100% is <code>1e18</code>, 50% is <code>5e17</code>, etc. These values are in the same format that the contract understands.</p> | 61762946000000000 |
| lpFeePct      | <p>The percent of the amount that will go to the LPs as a fee for borrowing their funds.<br><br>The formatting of the percentage is the same as <code>relayFeePct</code> .</p>                                                                                                                                                                                                                                                                                                                                                                      | 1252191895805000  |
| timestamp     | <p>The quote timestamp that was used to compute the <code>lpFeePct</code>.</p><p></p><p>To pay the quoted LP fee,  the user would need to pass this quote timestamp to the protocol when sending their bridge transaction.</p>                                                                                                                                                                                                                                                                                                                      | 1646925270        |

Errors:&#x20;

* 400: invalid input.
* 500: an unexpected error within the API.

### Querying Limits

The API uses the UMA SDK under the hood, but offers a convenient way to get transfer limits.

Example: Finding limits for bridging [USDC](https://optimistic.etherscan.io/token/0x7f5c764cbc14f9669b88837ca1490cca17c31607) from Optimism to Arbitrum.

You can visit this example in your browser: [Link](https://across.to/api/limits?token=0x7f5c764cbc14f9669b88837ca1490cca17c31607\&destinationChainId=42161)

Or curl it on the CLI:&#x20;

`curl "https://across.to/api/limits?token=0x7f5c764cbc14f9669b88837ca1490cca17c31607&destinationChainId=42161"`

#### API Definition

All API calls use `https://across.to/api`as the host.

#### limits

Path: `/limits`

Method: `GET`

Query Params

<table><thead><tr><th>Parameter Name</th><th width="260.3333333333333">Description</th><th>Example</th></tr></thead><tbody><tr><td>token</td><td>Address of token contract to transfer. For ETH (or other native tokens, like matic) use, use the wrapped address, like WETH.<br><br>Note: the address provided can be the token address on any chain. In the unlikely event where two different tokens have the same address on different chains, you can use the optional chainId parameter defined below to indicate which chain should be used.</td><td>0x7f5c764cbc14f9669b88837ca1490cca17c31607</td></tr><tr><td>destinationChainId</td><td>The intended destination of the transfer.</td><td>42161</td></tr><tr><td>originChainId (optional)</td><td>Used to specify which chain where the specified token address exists. Note: this is only needed to disambiguate when there are matching addresses on different chains. In that case, an arbitrary one will be chosen, so it is recommended that this is always provided.</td><td>10</td></tr></tbody></table>

Returns a JSON object with the following properties:

| Property Name        | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | Example        |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------- |
| minDeposit           | <p>The minimum deposit size in the tokens' units.</p><p></p><p>Note: USDC has <code>6</code> decimals, so this value would be the number of USDC multiplied by <code>1e6</code>. For WETH, that would be <code>1e18</code>.</p>                                                                                                                                                                                                                                                                                                                                                                                                                                               | 7799819        |
| maxDeposit           | <p>The maximum deposit size in the tokens' units.</p><p></p><p>Note: The formatting of this number is the same as minDeposit.</p>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | 22287428516241 |
| maxDepositInstant    | The max deposit size that can be relayed "instantly" on the destination chain. Instantly means that there is relayer capital readily available and that a relayer is expected to relay within 1-4 minutes of the deposit.                                                                                                                                                                                                                                                                                                                                                                                                                                                     | 201958902363   |
| maxDepositShortDelay | <p>The max deposit size that can be relayed with a "short delay" on the destination chain. This means that there is relayer capital available on mainnet and that a relayer will immediately begin moving that capital over the canonical bridge to relay the deposit. Depending on the chain, the time for this can vary. Polygon is the worst case where it can take between 20 and 35 minutes for the relayer to receive the funds and relay. Arbitrum is much faster, with a range between 5 and 15 minutes.<br><br>Note: if the transfer size is greater than this, the estimate should be between 2-4 hours for a slow relay to be processed from the mainnet pool.</p> | 2045367713809  |

Errors:&#x20;

* 400: invalid input.
* 500: an unexpected error within the API.

### Finding Available Routes

Example:

You can visit this example in your browser: [Link](https://across.to/api/available-routes)

Or curl it on the CLI:

`curl "https://across.to/api/available-routes"`

#### API Definition

All API calls use `https://across.to/api`as the host.

#### `available-routes`

Path: `/available-routes`

Method: `GET`

Query Params

| Property Name      | Description                                                                                                                                                                                                                                                                                                                              | Example                                    |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| originChainId      | <p>The chain ID of the originating chain to a bridge transfer. </p><p></p><p>Note: This is an <em>optional</em> query parameter. This parameter will filter the response JSON based. This filter can be used in addition to additional parameters to create a custom filter.</p>                                                         | 1                                          |
| destinationChainId | <p>The chain ID of the destination chain to a bridge transfer. </p><p></p><p>Note: This is an <em>optional</em> query parameter. This parameter will filter the response JSON based. This filter can be used in addition to additional parameters to create a custom filter.</p>                                                         | 10                                         |
| originToken        | <p>The token address of the originating bridge transfer. </p><p>Must be a valid ERC-20 token address. </p><p></p><p>Note: This is an <em>optional</em> query parameter. This parameter will filter the response JSON based. This filter can be used in addition to additional parameters to create a custom filter.</p>                  | 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2 |
| destinationToken   | <p>The token address that funds will be transferred to at the destination chain. Must be a valid ERC-20 token address. </p><p></p><p>Note: This is an <em>optional</em> query parameter. This parameter will filter the response JSON based. This filter can be used in addition to additional parameters to create a custom filter.</p> | 0x4200000000000000000000000000000000000006 |

Returns a JSON array of `Objects` with the following properties:

| Property Name      | Description                                                                                                                                   | Example                                    |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| originChainId      | The chain ID of the originating chain to a bridge transfer.                                                                                   | 1                                          |
| destinationChainId | The chain ID of the destination chain to a bridge transfer.                                                                                   | 10                                         |
| originToken        | <p>The token address of the originating bridge transfer. </p><p><br>Note: this will be a valid ERC-20 token address. </p>                     | 0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2 |
| destinationToken   | <p>The token address that funds will be transferred to at the destination chain.<br><br>Note: this will be a valid ERC-20 token address. </p> | 0x4200000000000000000000000000000000000006 |

Errors:&#x20;

* 400: invalid input.
* 500: an unexpected error within the API.
