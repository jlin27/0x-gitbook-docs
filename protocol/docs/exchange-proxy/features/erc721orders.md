---
description: ERC721<->ERC20/ETH Limit Order Functionality
---

# ERC721Orders

This feature implements the ERC721<->ERC20/ETH limit order type. The source code for [ERC721OrdersFeature.sol](https://github.com/0xProject/protocol/blob/c1177416f50c2465ee030dacc14ff996eebd4e74/contracts/zero-ex/contracts/src/features/nft\_orders/ERC721OrdersFeature.sol) can be found [here](https://github.com/0xProject/protocol/blob/c1177416f50c2465ee030dacc14ff996eebd4e74/contracts/zero-ex/contracts/src/features/nft\_orders/ERC721OrdersFeature.sol). \
\
Here are some highlights:

* Limit orders in the 0x protocol are off-chain messages that define a fixed-rate trade signed by a "maker" and filled on-chain by a "taker."
* The protocol is non-custodial in that neither party needs to deposit assets into the protocol in advance. Instead, _both parties must first_ [_grant the protocol_ ](../../../../introduction/0x-cheat-sheet.md)_an ERC20/ERC721 allowance for the assets they intend to sell_, and the protocol will spend this allowance during settlement.
* Due to the non-custodial nature of the protocol, the raw native asset (e.g., ETH) is only supported as a taker or fee token. Makers must use a wrapped variant (e.g., WETH) if trying to sell the network's native asset.
* ERC721 limit orders can only be filled once.
* Once a limit order is signed, anyone with the order and signature details can fill the order (unless the order restricts the `taker` address). There is no way to un-sign an order so cancellations must be performed on-chain before a taker attempts to fill it.
* An NFT order can be made for a ERC721 token with an unknown ID (see [ERC721 Buy Orders](erc721orders.md#erc721-buy-orders)).

### The ERC721 Limit Order Struct

```solidity
struct ERC721Order {
    TradeDirection direction;
    address maker;
    address taker;
    uint256 expiry;
    uint256 nonce;
    address erc20Token;
    uint256 erc20TokenAmount;
    Fee[] fees;
    address erc721Token;
    uint256 erc721TokenId;
    Property[] erc721TokenProperties;
}
```

* `direction`: Whether this order is selling an NFT or buying an NFT (see [#erc721-sell-orders](erc721orders.md#erc721-sell-orders "mention") or [#erc721-buy-orders](erc721orders.md#erc721-buy-orders "mention")).
* `maker`: The maker/signer of this limit order.
* `taker`: Who can fill this order. May be NULL (`0x000...`) to allow anyone.
* `expiry`: The block timestamp at which this trade is no longer valid.
* `nonce`: Usually a random nonce the maker sets to ensure the hash of this order is unique and to identify it during fills and cancellations. Carefully choosing this value can be used to reduce gas usage (see [#smart-nonces](erc721orders.md#smart-nonces "mention")).
* `erc20Token`: The ERC20 token (or native asset, if allowed) to exchange for the NFT. For eligible trades, the native asset may be specified as `0xeee...`.
* `erc20TokenAmount`: The amount of `erc20Token` to exchange for the NFT.
* `fees`: Optional fees, denominated in `erc20Token`, that are paid out to designated recipients when the order is filled. Fees are collected _in addition_ to the `erc20TokenAmount`.
* `erc721Token`: The NFT's ERC721 token contract address being exchanged.
* `erc721TokenId`: The ID of the ERC721 token inside `erc721Token` being exchanged. This will be ignored for [buy orders](erc721orders.md#erc721-buy-orders) with valid `erc721TokenProperties`.
* `erc721TokenProperties`: A series of contracts that will be called to validate an unknown NFT token ID being bought at settlement.

Where `TradeDirection` is:

```solidity
enum TradeDirection {
    SELL_NFT,
    BUY_NFT
}
```

`Fee` is:

```solidity
struct Fee {
    address recipient;
    uint256 amount;
    bytes feeData;
}
```

`Property` is:

```solidity
struct Property {
    address propertyValidator;
    bytes propertyData;
}
```

### Order Hashes

The protocol utilizes a canonical, [EIP712](https://eips.ethereum.org/EIPS/eip-712) hash of the limit order struct to verify the authenticity of the order. In typical usage, this is the hash that the maker signs to make the order valid and fillable.

Computing the hash of a limit order in solidity is given by:

```solidity
bytes32[] memory feesStructHashArray = new bytes32[](order.fees.length);
for (uint256 i = 0; i < order.fees.length; ++i) {
    feesStructHashArray[i] = keccak256(abi.encode(
        bytes32(0xe68c29f1b4e8cce0bbcac76eb1334bdc1dc1f293a517c90e9e532340e1e94115),
        order.fees[i].recipient,
        order.fees[i].amount,
        keccak256(order.fees[i].feeData)
    ));
}
bytes32 memory feesHash = keccak256(abi.encodePacked(
    feesStructHashArray
));
bytes32[] memory propertiesStructHashArray = new bytes32[](order.erc721TokenProperties.length);
for (uint256 i = 0; i < order.erc721TokenProperties.length; ++i) {
    propertiesStructHashArray[i] = keccak256(abi.encode(
        bytes32(0x6292cf854241cb36887e639065eca63b3af9f7f70270cebeda4c29b6d3bc65e8),
        order.erc721TokenProperties[i].propertyValidator,
        keccak256(order.erc721TokenProperties[i].propertyData)
    ));
}
bytes32 memory propertiesHash = keccak256(abi.encodePacked(
    propertiesStructHashArray
));
keccak256(abi.encode(
    bytes32(0x2de32b2b090da7d8ab83ca4c85ba2eb6957bc7f6c50cb4ae1995e87560d808ed),
    order.direction,
    order.maker,
    order.taker,
    order.expiry,
    order.nonce,
    order.erc20Token,
    order.erc20TokenAmount,
    kecca256(abi,
    order.erc721Token,
    order.erc721TokenId,
    propertiesHash
));
```

Another way to get the hash of an order is to simply call `getERC721OrderHash()` on the [Exchange Proxy](../).

### Signatures

To fill an order, the taker must (usually, see below) also submit an ECDSA signature generated for that order, typically signed by the maker. For information on the format an how to generate these signatures, see [signatures.md](../../signatures.md "mention").

### Smart Contract Makers

Smart contracts do not have a private key associated with them so they cannot generate a valid ECDSA signature. Yet they still can operate as the `maker` of an order. To do this, they must call `preSignERC721Order()`, which will allow the order to be filled using the `PRESIGNED` `signatureType`.

### Network Discoverable Orders

Another feature of the `preSignERC721Order()` function is that it will emit an event `ERC721OrderPreSigned` (see [#events](erc721orders.md#events "mention"))  listing all the details of the order. Anyone can listen for these events and potentially fill the order without relying on off-chain sharing mechanisms.

### Smart Nonces

The `nonce` field in the order is a number the maker chooses to ensure uniqueness of the order, but it also has some other functions:

* To identify the order (in addition to the caller/maker) in cancellation functions (see [#maker-functions](erc721orders.md#maker-functions "mention")).
* To identify the order when checking/updating its filled state.
* To potentially reuse EVM storage slots that mark an order as filled, which can provide significant gas savings (\~15k) when filling or cancelling the order.

#### Gas Optimized Nonces

The protocol marks ERC721 orders as filled when they either get cancelled or filled. Instead of always using a completely new storage slot for each order from a maker, the upper 248 bits of the nonce will identify the storage slot/bucket/status vector to be used and the lower 8 bits of the nonce will identify the bit offset in that slot (each slot is also 256 bits) which will be flipped from 0 to 1 when the order is cancelled or filled.

![](../../../../.gitbook/assets/smart-nonces.png)

To take advantage of this gas optimization, makers should reuse the upper 248 bits of their nonce across up to 256 different orders, varying the value of the lower 8 bits between them.

### ERC721 Sell Orders

Sell orders are orders where `direction` is set to `TradeDirection.SELL_NFT`, which indicates that a _maker wishes to **sell** an ERC721 token that they possess_.

For these orders, the maker must set the `erc721Token` and `erc721TokenId` fields to the ERC721 token contract address  and the ID of the  ERC721 token they're trying to sell, respectively. The `erc20Token` and `erc20TokenAmount` fields will be set to the ERC20 token and the ERC20 amount they wish to receive for their NFT.

### ERC721 Buy Orders

Buy orders are where `direction` is set to `TradeDirection.BUY_NFT`, which indicates that a _maker wishes to **buy** an ERC721 token that they do not possess_.

A buy order can be created for a known token ID or an unknown token ID.

#### Buying a Known NFT Token ID

This is when the maker knows exactly which NFT they want, down to the token ID. These are fairly straight-forward and just require the maker setting the `erc721Token` and `erc721TokenId` fields to the ERC721 token contract address and the ID of the token they want to buy, respectively. The The `erc20Token` and `erc20TokenAmount` fields will be set to the ERC20 token and the ERC20 amount they are willing to pay for that particular NFT. The `erc721TokenProperties` field should be set to the empty array.

#### Buying an Unknown NFT Token ID

This is when the maker will accept any token ID from an ERC721 contract. In this scenario, the `erc721TokenProperties` order field is used to signal and validate terms of this kind of order.

If a maker wishes to accept any token ID, regardless of any other properties of that NFT, they should set the `erc721TokenProperties` field to an array with exactly one item in it:

```solidity
[
    Property({
        propertyValidator: address(0),
        propertyData: ""
    })
]
```

If a maker wishes to accept any token ID, but only if that token also satisfies some other properties (e.g., cryptokitty with green eyes), they should set `erc721TokenProperties` to an array of callable property validator contracts. Each `propertyValidator` contract must implement the `IPropertyValidator` interface, which is defined as:

```solidity
interface IPropertyValidator {
    function validateProperty(
        address tokenAddress,
        uint256 tokenId,
        bytes calldata propertyData
    )
        external
        view;
}

```

Each entry in `erc721TokenProperties` with a non-null `propertyValidator` address will be called using this interface, passing in the `tokenAddress` and `tokenId` of the ERC721 the taker is trying to fill the order with. If the validator fails to validate the requested properties of the NFT, it should revert.

The `propertyData` field of each property entry is supplied by the maker and is also passed in.  For example, if a property validator contract was designed to check that a cryptokitty had a specific eye color, the `propertyData` could simply be some encoding of `"green"`, and the validator contract would parse this data and perform the necessary checks on the token to ensure it has green eyes.

### Taker Callbacks

Some fill functions accept a `callbackData` `bytes` parameter. If this parameter is non-empty (`length > 0`), a call will be issued against the caller using `callbackData` as the full call data. This call happens after the transfer of the maker asset and before the transfer of the taker asset. This allows a smart contract taker to leverage the maker's asset to fulfill their end of the trade. For example, if a taker finds a compatible, complementary order on another protocol they could fulfill that order with the maker's asset, supply the Exchange Proxy with the necessary opposing asset with the proceeds, and pocket the difference.

### Information Functions (read-only)

* **`validateERC721OrderSignature(ERC721Order order, Signature signature)`**
  * Check if `signature` is valid for `order`. Reverts if not.&#x20;
* **`validateERC721OrderProperties(ERC721Order order, uint256 erc721TokenId)`**
  * Validates that a given `erc721TokenId` satisifes a _buy_ order.
  * For orders with a known token ID, verifies that they match.
  * For orders with an unknown token ID, checks that all property validators succeed for the token.
  * Reverts if `erc721TokenId` does not satisfy the buy order.
* **`getERC721OrderStatus(ERC721Order order) returns (OrderStatus status)`**
  * Get the status of an ERC721 order. `status` will be one of: `INVALID (0)`, `FILLABLE (1)`, `UNFILLABLE (2)`, or `EXPIRED (3)`.
* **getERC721OrderStatusBitVector(address maker, uint248 nonceRange) returns (uint256 bitVector)**
  * Retrieve the fillable status word given a maker and a storage bucket defined in an order nonce. This will be a bit vector of filled/cancelled states for all orders by `maker` with a nonce whose upper 248 bits equal `nonceRange`. See [#smart-nonces](erc721orders.md#smart-nonces "mention") for more details.

### Maker Functions

* **`preSignERC721Order(ERC721Order order)`**
  * If called by `order.maker`, marks the order as presigned, which means takers can simply pass in an empty signature with `signatureType = PRESIGNED`.
  * Must be called by the `order.maker`.
  * This can also be used to publish/broadcast orders on-chain, as it will emit an `ERC721OrderPreSigned` event.
* **`cancelERC721Order(uint256 orderNonce)`**
  * Cancels any order with `nonce == orderNonce` and where the `maker` is the caller.
* **`batchCancelERC721Orders(uint256[] orderNonces)`**
  * Batch version of `cancelERC721Order()`.

### Taker/Fill Functions

* **`sellERC721(ERC721Order buyOrder, Signature signature, uint256 erc721TokenId, bool unwrapNativeToken, bytes callbackData)`**
  * Sell an ERC721 token `erc721TokenId` to a `buyOrder`.
  * The `signature` must be a valid signature for `buyOrder` generated by `buyOrder.maker`.
  * If `unwrapNativeToken` is `true` and the `erc20Token` of the order is a wrapped version of the native token (e.g., WETH), this will also unwrap and transfer the native token to the taker.
  * If `callbackData` is non-empty, the taker (`msg.sender`) will be called using this `callbackData` as call data. This happens after `erc20Token` is transferred to the taker, but before `erc721TokenId` is transferred to the maker.
* **`buyERC721(ERC721Order sellOrder, Signature signature, bytes callbackData) payable`**
  * Buy an ERC721 token being sold by `sellOrder`.
  * The `signature` must be a valid signature for `sellOrder` generated by `sellOrder.maker`.
  * If the order's `erc20Token` is the the native token (`0xeee...`), the native token (ETH on Ethereum) must be attached to the function call.&#x20;
  * If `callbackData` is non-empty, the taker (`msg.sender`) will be called using this `callbackData` as call data. This happens after the ERC721 token is transferred to the taker, but before `erc20Token` is transferred to the maker.
* **`batchBuyERC721s(ERC721Order[] sellOrders, Signature[] signatures, bytes[] callbackData, bool revertIfIncomplete) payable returns (bool[] successes)`**
  * Batch version of `buyERC721()`.
  * If `revertIfIncomplete` is true, reverts if any of the individual buys fail.
  * If `revertIfIncomplete` is `false`, returns an array of which respective fill succeeded.
* **`matchERC721Orders(ERC721Order sellOrder, ERC721Order buyOrder, Signature sellOrderSignature, Signature buyOrderSignature) returns (uint256 profit)`**
  * Matches two complementary ERC721 buy and sell orders.
  * The `sellOrder` must be willing to receive at least the `erc20Token` amount `buyOrder` is offering.
  * The NFT being sold by `sellOrder` must be compatible with the requirements of `buyOrder`.
  * Returns the spread between the two orders (less fees), which will go to the caller of this function.
  * If any fees are involved in either order, they will be taken from `profit`.
* **`batchMatchERC721Orders(ERC721Order[] sellOrders, ERC721Order[] buyOrders, Signature[] sellOrderSignatures, Signature[] buyOrderSignatures ) returns (uint256[] profits, bool[] successes)`**
  * Batch version of `matchERC721Orders()`.
  * Returns the spread of each respective match operation (less fees).
  * If `revertIfIncomplete` is `true`, reverts if any of the individual match operations fail.
  * If `revertIfIncomplete` is `false`, returns an array of which respective operation succeeded.

### Events

```solidity
/// @dev Emitted whenever an `ERC721Order` is filled.
/// @param direction Whether the order is selling or
///        buying the ERC721 token.
/// @param maker The maker of the order.
/// @param taker The taker of the order.
/// @param nonce The unique maker nonce in the order.
/// @param erc20Token The address of the ERC20 token.
/// @param erc20TokenAmount The amount of ERC20 token
///        to sell or buy.
/// @param erc721Token The address of the ERC721 token.
/// @param erc721TokenId The ID of the ERC721 asset.
/// @param matcher If this order was matched with another using `matchERC721Orders()`,
///                this will be the address of the caller. If not, this will be `address(0)`.
event ERC721OrderFilled(
    LibNFTOrder.TradeDirection direction,
    address maker,
    address taker,
    uint256 nonce,
    IERC20TokenV06 erc20Token,
    uint256 erc20TokenAmount,
    IERC721Token erc721Token,
    uint256 erc721TokenId,
    address matcher
);

/// @dev Emitted whenever an `ERC721Order` is cancelled.
/// @param maker The maker of the order.
/// @param nonce The nonce of the order that was cancelled.
event ERC721OrderCancelled(
    address maker,
    uint256 nonce
);

/// @dev Emitted when an `ERC721Order` is pre-signed.
///      Contains all the fields of the order.
event ERC721OrderPreSigned(
    LibNFTOrder.TradeDirection direction,
    address maker,
    address taker,
    uint256 expiry,
    uint256 nonce,
    IERC20TokenV06 erc20Token,
    uint256 erc20TokenAmount,
    LibNFTOrder.Fee[] fees,
    IERC721Token erc721Token,
    uint256 erc721TokenId,
    LibNFTOrder.Property[] erc721TokenProperties
);
```
