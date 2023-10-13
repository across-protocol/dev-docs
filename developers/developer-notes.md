# Developer notes

This page includes a running list of Across behaviors that developers who are using Across should be aware of:

### ETH/WETH Behavior

Across liquidity pools are filled using WETH but, depending on the context, Across will sometimes send a user ETH and sometimes send a user WETH.

* If a bridge transfer is being sent to an EOA, **the EOA will receive ETH** (not WETH)
* If a bridge transfer is being sent to a contract, **the contract will receive WETH** (not ETH)

### More questions?

If you are a developer working on integrating your project with Across, please reach out to us on [Discord](http://discord.across.to/)!

We look forward to helping you integrate with the best bridge around!
