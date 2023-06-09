---
description: Consume 0x API swap quotes from inside a smart contract.
---

# Use 0x API Liquidity in Your Smart Contracts

## Overview

After you complete this guide, you will have a smart contract that swaps WETH for DAI, powered by 0x API.

In the example, we will:

1. Deposit WETH into our contract.
2. Fetch a quote to sell WETH for DAI from 0x-API.
3. Fill that quote through our contract, converting its WETH into DAI.

## Set up the project

We will use our [0x API starter project](https://github.com/0xProject/0x-api-starter-guide-code), which contains the all the contracts and javascript necessary for this example. This project will use [`truffle`](https://www.trufflesuite.com/truffle) for compiling and deploying our contracts and [`web3.js`](https://web3js.readthedocs.io/en/v1.3.0/) for interacting with deployed contracts.

Begin by cloning the starter project and installing dependencies:

```bash
$ git clone git@github.com:0xProject/0x-api-starter-guide-code
$ cd 0x-api-starter-guide-code
$ npm install --dev
```

### The forked test environment

All the examples in the project can be run without cost using a forked [ganache](https://github.com/trufflesuite/ganache-cli) instance against Ethereum mainnet. To start the forked instance, run the following:

```bash
# Replace ETHEREUM_RPC_URL with your mainnet node endpoint URL (e.g., infura)
$ RPC_URL=ETHEREUM_RPC_URL npm run start-fork
```

## The swap contract

The contract that will be directly filling 0x-API quotes is [`SimpleTokenSwap`](https://github.com/0xProject/0x-api-starter-guide-code/blob/master/contracts/SimpleTokenSwap.sol). It implements a `fillQuote()` function that accepts and executes a 0x-API quote to convert some amount of its ERC20 tokens into another. For simplicity, this contract is not designed for use with plain ETH. However, WETH and ETH pairs are identical markets in 0x-API, so this will not affect pricing.

### The fill function

The swap is executed when the owner calls the `fillQuote()` function. Prior to calling this function, the `SimpleTokenSwap` contract should be funded with at least `sellAmount` (from the API response) of `sellToken` to complete the swap.

In this function we:

1. Grant the `allowanceTarget` an allowance to spend `sellToken` on this contract's behalf.
2. Execute the ERC20->ERC20 swap.
3. Transfer any leftover ETH (protocol fee refunds) to the sender.

```solidity
// Swaps ERC20->ERC20 tokens held by this contract using a 0x-API quote.
function fillQuote(
    // The `sellTokenAddress` field from the API response.
    IERC20 sellToken,
    // The `buyTokenAddress` field from the API response.
    IERC20 buyToken,
    // The `allowanceTarget` field from the API response.
    address spender,
    // The `to` field from the API response.
    address payable swapTarget,
    // The `data` field from the API response.
    bytes calldata swapCallData
)
    external
    onlyOwner
    payable // Must attach ETH equal to the `value` field from the API response.
{
    // ...

    // Give `spender` an infinite allowance to spend this contract's `sellToken`.
    // Note that for some tokens (e.g., USDT, KNC), you must first reset any existing
    // allowance to 0 before being able to update it.
    require(sellToken.approve(spender, uint256(-1)));
    // Call the encoded swap function call on the contract at `swapTarget`,
    // passing along any ETH attached to this function call to cover protocol fees.
    (bool success,) = swapTarget.call{value: msg.value}(swapCallData);
    require(success, 'SWAP_CALL_FAILED');
    // Refund any unspent protocol fees to the sender.
    msg.sender.transfer(address(this).balance);

    // ...
}
```

### Deposit function

Since the contract will sell its own balance of a token when executing a swap, it needs to hold a balance of that token beforehand. In this example, we will be selling WETH, which can easily be minted from ETH. So the contract has a deposit function that accepts ETH and wraps it into WETH.

```solidity
// Transfer ETH into this contract and wrap it into WETH.
function depositETH()
    external
    payable
{
    WETH.deposit{value: msg.value}();
}
```

### Payable fallback

Certain quotes require a protocol fee, in ETH, to be attached to the swap call. But it's possible that by the time the transaction is mind, the swap will end up not needing to pay the protocol fee. In these cases, the protocol fee would be refunded to the taker, which is in this case is the `SimpleTokenSwap` contract. So it's important that your contract be able to receive this ETH through either a payable `fallback()` or `receive()` function (in solidity 0.6+).

```solidity
// Payable fallback to allow this contract to receive protocol fee refunds.
receive() external payable {}
```

### Compiling and deploying the contract

Before we can run the example, we will need to compile and deploy the contract to the forked network:

```bash
npm run deploy-fork
```

## Using the contract

The complete interaction with the contract is in the [`swap-contract.js`](https://github.com/0xProject/0x-api-starter-guide-code/blob/master/src/swap-contract.js) script. This script does the following:

* Fund the deployed `SimpleTokenSwap` contract with WETH.
* Fetch a swap quote from 0x-API to convert said WETH to DAI.
* Call `fillQuote()` on the `SimpleTokenSwap` contract to execute the swap.

### Fund the contract

First we call `depositETH()`, which accepts ETH and wraps it into WETH, to give the contract a balance (`sellAmountWei`) of WETH .

```js
await waitForTxSuccess(contract.methods.depositETH().send({
    value: sellAmountWei,
    from: owner,
}));
```

### Fetch a quote from 0x API

Next we make an HTTP request to 0x-API (at `https://api.0x.org/swap/v1/quote`) for a quote to convert that WETH into DAI.

```js
const qs = createQueryString({
    sellToken: 'WETH',
    buyToken: 'DAI',
    sellAmount: sellAmountWei,
});
const quoteUrl = `${API_QUOTE_URL}?${qs}`;
const response = await fetch(quoteUrl);
const quote = await response.json();
```

The returned quote will have fields such as:

```js
{
    to: '0xdef1c0ded9bec7f1a1670819833240f027b25eff',
    data: '0x415565b0000000000000000000000000...',
    allowanceTarget: '0xf740b67da229f2f10bcbd38a7979992fcc71b8eb',
    sellTokenAddress: '0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2',
    buyTokenAddress: '0x6b175474e89094c44da98b954eedeac495271d0f',
    value: '0',
    gasPrice: '8000000000',
    gas: 12345678,
    ...
}
```

### Fill the quote through our contract

We can now pass fields from this quote into the `fillQuote()` function on our contract to be filled. This will cause the contract to swap its WETH balance for DAI.

```js
const receipt = await waitForTxSuccess(contract.methods.fillQuote(
        quote.sellTokenAddress,
        quote.buyTokenAddress,
        quote.allowanceTarget,
        quote.to,
        quote.data,
    ).send({
        from: owner,
        value: quote.value,
        gasPrice: quote.gasPrice,
    }));
```

### Running the example

We can run this example with the command:

```bash
npm run swap-contract-fork
```

If all goes well, you should see output similar to:

```
Fetching quote https://api.0x.org/swap/v1/quote?sellToken=WETH&buyToken=DAI&sellAmount=100000000000000000...
Received a quote with price 371.15289757586245518
Filling the quote through the contract at 0x7382949f535C1bb4D64059b934d4A63A11D3DAa2...
✔ Successfully sold 0.1 WETH for 37.115276285540685651 DAI!
```

## Next steps:

* Refer to our [0x API swap specification](../../0x-orderbook-api/api-references/) for detailed documentation
* If you need help building on 0x, please use[ Ethereum StackExchange](https://ethereum.stackexchange.com/questions/tagged/0x) with the '0x' tag so we're properly notified.&#x20;
