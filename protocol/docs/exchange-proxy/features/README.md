---
description: Documentation on the individual features of the Exchange Proxy
---

# Features

This is a non-exhaustive overview of commonly referenced features that may be registered/deployed on the Exchange Proxy. What features an instance of the Exchange Proxy has depends on the needs of the network it is deployed on. For instance, limit orders are not typically deployed on networks where the only focus for the protocol is swap aggregation.\
\
For a deep dive into all the features of the 0x Protocol, check out the [0x Protocol documentation](https://protocol.0x.org/en/latest/).&#x20;

* [**NativeOrdersFeature**](nativeorders.md)
  * Settlement and management of ERC20 Limit and RFQ Orders.
* **BatchFillNativeOrdersFeature**
  * Exposes batch fill functions for ERC20 Limit and RFQ Orders.
* [**ERC721OrdersFeature**](erc721orders.md) **/** [**ERC1155OrdersFeature**](erc1155orders.md)
  * Settlement and management of NFT Limit Orders.
* **TransformERC20Feature**
  * Primary fulfillment function for 0x API swap quotes.
* **MultiplexFeature**
  * Gas-optimized fulfillment functions for a subset of 0x API swap quotes.
* **LiquidityProviderFeature**
  * Fulfillment functions for liquidity sources implementing the `ILiquidityProvider` interface.
* **MetaTransactionsFeature**
  * Allows one to executes operations within the Exchange Proxy on behalf of another.
* **OtcOrdersFeature**
  * &#x20;Settlement functions for a more restrictive form of RFQ orders.
* **UniswapFeature**
  * Fulfillment function for swap 0x API quotes against Uniswap V2 and some forks of it (Ethereum mainnet only).
* **PancakeSwapFeature**
  * Fulfillment function for swap 0x API quotes against Pancakeswap (BSC only).&#x20;
* **SimpleFunctionRegistryFeature**
  * Administrative functions for managing the feature/function registry.
* **OwnableFeature**
  * Administrative functions for feature migrations.
* **FundRecoveryFeature**
  * Administrative functions for rescuing tokens that have been mistakenly sent to the Exchange Proxy.
