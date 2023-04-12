---
description: ERC20 Limit Orders Feature of the Exchange Proxy
---

# NativeOrders

## Limit Orders

Highlights:

* Limit orders in the 0x protocol are off-chain messages that define a fixed-rate trade signed by a "maker" and filled on-chain by a "taker."
* The protocol is non-custodial in that neither party needs to deposit funds into the protocol in advance. Instead, _both parties must first_ [_grant the protocol_](../../../../introduction/0x-cheat-sheet.md) _an ERC20 allowance for the tokens they intend to sell_, and the protocol will spend this allowance during settlement.
* The exchange rate of a limit order is given by the taker and maker token quantities.
* Limit orders may be repeatedly partially filled until the order is fully filled (the full quantities of either taker and maker tokens defined in the order have been exchanged through the order).
* Once a limit order is signed, anyone with the order and signature details can fill the order (unless the order restricts the `taker` or `sender` address). There is no way to un-sign an order so cancellations must be performed on-chain before a taker attempts to fill it.
* ERC20 limit orders only work with ERC20 tokens, meaning raw (unwrapped) native asset (ETH on Ethereum) cannot be used as a maker or taker token. E.g., on Ethereum, WETH must be used instead.

### The Limit Order Struct

```solidity
struct LimitOrder {
    address makerToken;
    address takerToken;
    uint128 makerAmount;
    uint128 takerAmount;
    uint128 takerTokenFeeAmount;
    address maker;
    address taker;
    address sender;
    address feeRecipient;
    bytes32 pool;
    uint64 expiry;
    uint256 salt;
}
```

* `makerToken`: The address of the ERC20 token being sold by the maker of this order and bought by the taker of this order.
* `takerToken`: The address of the ERC20 token being sold by the taker of this order and bought by the maker of this order.
* `makerAmount`: The total quantity of `makerToken` that can be bought/sold by when filling this order. The ratio of this with `takerToken` establish the exchange rate.
* `takerAmount`: The total quantity of `takerToken` that can be bought/sold by when filling this order. The ratio of this with `takerToken` establish the exchange rate.
* `takerTokenFeeAmount`: May be `0`. The total fee that can be paid to `feeRecipient` when filling this order, denominated in `takerToken`. Fees are paid _in addition_ to the `takerAmount` when the order is filled.
* `maker`: The maker of this order. This is the Ethereum address that creates and signs the order.
* `taker`: May be NULL (`0x000...`). If set, only this address can fill this order.
* `sender`: May be NULL (`0x000...`). If set, only this address can directly call `fillLimitOrder()`. This is intended for use with meta-transactions.
* `feeRecipient`: The recipient of any fees determined by `takerTokenFeeAmount` when the order is filled.
* `pool` (defunct): The staking pool that receives a the protocol fee paid for filling this order. Currently protocol fees are disabled so this does nothing.
* `expiry`: The block timestamp (UTC) of when this order expires. The order cannot be filled at or after this time.
* `salt`: A random nonce the maker sets to ensure the hash of this order is unique. Some makers choose to use an montonically increasing value for this field to take advantage of bulk cancellation functions.

### Limit Order Hashes

The protocol utilizes a canonical, [EIP712](https://eips.ethereum.org/EIPS/eip-712) hash of the limit order struct to identify the order internally. In typical usage, this is also the hash that the maker signs to make the order valid and fillable.

Computing the hash of a limit order in solidity is given by:

```solidity
keccak256(abi.encode(
    bytes32(0xce918627cb55462ddbb85e73de69a8b322f2bc88f4507c52fcad6d4c33c29d49),
    order.makerToken,
    order.takerToken,
    order.makerAmount,
    order.takerAmount,
    order.takerTokenFeeAmount,
    order.maker,
    order.taker,
    order.sender,
    order.feeRecipient,
    order.pool,
    order.expiry,
    order.salt
));
```

Another way to get the hash of an order is to simply call `getLimitOrderHash()` on the [Exchange Proxy](../).

### Limit Order Signatures

To fill an order, the taker must also submit an ECDSA signature generated for that order, typically signed by the maker. For information on the format an how to generate these signatures, see [signatures.md](../../signatures.md "mention").

### Smart Contract Makers

Smart contracts do not have a private key associated with them so they cannot generate a valid ECDSA signature. Yet they still can operate as the `maker` of an order. To do this, they must delegate an external signer address who can sign and cancel orders (using the above signature schemes) on their behalf. See `registerAllowedOrderSigner()`.

### Information Functions

* **`getLimitOrderInfo(LimitOrder order) view returns (OrderInfo memory orderInfo)`**
  * Get order info for a limit order.
  * Returns a struct with the `orderHash`, `status`, and `takerTokenFilledAmount` (how much the order has been filled, denominated in taker tokens).
* **`getLimitOrderHash(LimitOrder order) view returns (bytes32 orderHash)`**
  * Get the [canonical hash](nativeorders.md#limit-order-hashes) of a limit order.
* **`isValidOrderSigner(address maker, address signer) view returns (bool isAllowed)`**
  * Check whether a `maker` has delegated an external `signer` address to sign orders on their behalf.

### Maker Functions

* **`registerAllowedOrderSigner(address[] origins, bool allowed)`**
  * Toggles whether multiple external addresses are allowed to sign orders on behalf of the caller of this function.
* **`cancelLimitOrder(LimitOrder order)`**
  * Mark an order cancelled for which the `maker` is the caller.
  * Succeeds even if the order is already cancelled.
* **`batchCancelLimitOrders(LimitOrder[] orders)`**
  * Batch version of `cancelLimitOrder()`.
* **`cancelPairLimitOrders(address makerToken, address takerToken, uint256 minValidSalt)`**
  * Cancels all limit orders for whom the `maker` is also the caller, with a given `makerToken`, `takerToken`, and a `salt < minValidSalt`.
* **`batchCancelPairLimitOrders(address[] makerTokens, address[] takerTokens, uint256[] minValidSalts)`**
  * Batch version of `cancelPairLimitOrders()`.
* **`cancelPairLimitOrdersWithSigner(address maker, address makerToken, address takerToken, uint256 minValidSalt)`**
  * Cancels all limit orders for with a given `maker`, `makerToken`, `takerToken`, and a `salt < minValidSalt`.
  * The caller must have previously been registered as an external order signer with `registerAllowedOrderSigner()`.
* **`batchCancelPairLimitOrdersWithSigner(address maker, address[] memory makerTokens, address[] takerTokens, uint256[] minValidSalts)`**
  * Batch version of `cancelPairLimitOrdersWithSigner()`.

### Taker/Fill Functions

* **`fillLimitOrder(LimitOrder order, Signature signature, uint128 takerTokenFillAmount) returns (uint128 takerTokenFilledAmount, uint128 makerTokenFilledAmount)`**
  * Fills a single limit order for _up to_ `takerTokenFillAmount` taker tokens. If less than `takerTokenFillAmount` tokens are left on the order (possibly due to a prior partial fill), the remaining amount will be filled instead.
  * The caller of this function is considered the taker.
  * `signature` is an ECDSA order signature generated by either the order's `maker` or a designated external order signer (see [#smart-contract-makers](nativeorders.md#smart-contract-makers "mention")).
  * Prior to calling this function, the maker and taker (caller) must have granted the Exchange Proxy an adequent ERC20 allowance for their respective token being sold.
  * Returns how much of both taker and maker tokens were actually filled during settlement.
* **`fillOrKillLimitOrder(LimitOrder order, Signature signature, uint128 takerTokenFillAmount) returns (uint128 makerTokenFilledAmount)`**
  * Similar to `fillLimitOrder()` but must fill _exactly_ `takerTokenFillAmount` taker tokens or else the function will revert.

### Events

```solidity
/// @dev Emitted whenever a `LimitOrder` is filled.
/// @param orderHash The canonical hash of the order.
/// @param maker The maker of the order.
/// @param taker The taker of the order.
/// @param feeRecipient Fee recipient of the order.
/// @param takerTokenFilledAmount How much taker token was filled.
/// @param makerTokenFilledAmount How much maker token was filled.
/// @param protocolFeePaid How much protocol fee was paid.
/// @param pool The fee pool associated with this order.
event LimitOrderFilled(
    bytes32 orderHash,
    address maker,
    address taker,
    address feeRecipient,
    address makerToken,
    address takerToken,
    uint128 takerTokenFilledAmount,
    uint128 makerTokenFilledAmount,
    uint128 takerTokenFeeFilledAmount,
    uint256 protocolFeePaid,
    bytes32 pool
);

/// @dev Emitted whenever a limit or RFQ order is cancelled.
/// @param orderHash The canonical hash of the order.
/// @param maker The order maker.
event OrderCancelled(
    bytes32 orderHash,
    address maker
);

/// @dev Emitted whenever Limit orders are cancelled by pair by a maker.
/// @param maker The maker of the order.
/// @param makerToken The maker token in a pair for the orders cancelled.
/// @param takerToken The taker token in a pair for the orders cancelled.
/// @param minValidSalt The new minimum valid salt an order with this pair must
///        have.
event PairCancelledLimitOrders(
    address maker,
    address makerToken,
    address takerToken,
    uint256 minValidSalt
);

/// @dev Emitted when new order signers are registered
/// @param maker The maker address that is registering a designated signer.
/// @param signer The address that will sign on behalf of maker.
/// @param allowed Indicates whether the address should be allowed.
event OrderSignerRegistered(
    address maker,
    address signer,
    bool allowed
);
```

## Relevant Code

* [`INativeOrdersFeature.sol`](https://github.com/0xProject/protocol/blob/development/contracts/zero-ex/contracts/src/features/interfaces/INativeOrdersFeature.sol)
* [`LibNativeOrders.sol`](https://github.com/0xProject/protocol/blob/development/contracts/zero-ex/contracts/src/features/libs/LibNativeOrder.sol)
* [`LibSignature.sol`](https://github.com/0xProject/protocol/blob/development/contracts/zero-ex/contracts/src/features/libs/LibSignature.sol)
* &#x20;[`NativeOrdersFeature.sol`](https://github.com/0xProject/protocol/blob/development/contracts/zero-ex/contracts/src/features/NativeOrdersFeature.sol)
