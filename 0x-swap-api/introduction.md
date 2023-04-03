---
description: >-
  An introduction to how the 0x Swap API works and the resources included this
  section
---

# Introduction

## What is 0x Swap API?

The 0x API is a REST API that runs on HTTP. The 0x API’s `/swap` endpoint aggregates liquidity from across the DEX ecosystem and easily surfaces the best price for ERC20 assets. The endpoint aggregates liquidity from both on-chain and off-chain liquidity networks.&#x20;

{% hint style="info" %}
📍 **New to 0x Swap API?** Watch this video on [🛠 Building Token Swaps with 0x API](https://www.youtube.com/watch?v=APXjSGLaoRw\&list=PLXzKMXK2aHh5oYMSL2stEUhgzgdbb58uV\&index=16). It covers and overview of how the Swap API works, how to use it, and project ideas.&#x20;
{% endhint %}

## Why use 0x Swap API?&#x20;

0x Swap API watches out for your users and is easy for developers. &#x20;

* By default, it has **built-in user-protection features**, including:
  * [Slippage Protection](https://docs.0x.org/0x-api-swap/advanced-topics/slippage-protection) which **protects** users **against MEV-attacks**
  * [Price Impact Protection](advanced-topics/price-impact-protection.md) which **protects** users **from getting rekt** by illiquid markets
* 0x collects limited [trade surplus](../developer-resources/faqs-and-troubleshooting.md) (read more [here](../developer-resources/faqs-and-troubleshooting.md#if-the-difference-between-the-quoted-price-and-the-executed-price-is-positive-what-happens-to-the-fu)).&#x20;
* It is **easy-to-use!** For example, you can easily find the best price to buy DAI with WETH with this request

{% hint style="info" %}
**(click to see live price response 👉)** [https://api.0x.org/swap/v1/price?buyToken=DAI\&sellToken=WETH\&sellAmount=100000000000000000](https://api.0x.org/swap/v1/quote?buyToken=DAI\&sellToken=WETH\&sellAmount=100000000000000000)
{% endhint %}

The response that is returned back is a valid unsigned Ethereum transaction that can be submitted _directly_ to a node to complete the swap. &#x20;

## How does it work?

\
Under the hood, the API performs certain tasks:

* **Queries prices** from multiple DEXs and market makers and **aggregates the liquidity** from the queried sources to provide the best price possible. Think of how Google flights aggregates  flight prices for a certain time and date to help you find the best price, `/swap` similarly helps you find the best price across DeFi liquidity sources.&#x20;
* Swap API’s **smart order routing** algorithm splits up your transaction across different sources to maximize the overall return on your swap. Read more about smart order routing [here](https://blog.0xproject.com/0x-apis-smart-order-routing-7af0195515e5).&#x20;
* Swap API's **** [**Slippage Protection** ](https://docs.0x.org/0x-api-swap/advanced-topics/slippage-protection)enables developers to surface more reliable quotes and consistently deliver the best executed price to users.
* Swap API's **** [**Price Impact Protection**](advanced-topics/price-impact-protection.md) calculates price impact estimates, and allows developers to easily notify users if insufficient liquidity may negatively affect the price.&#x20;
* 0x API's[ **** ](https://docs.0x.org/0x-api-swap/advanced-topics/slippage-protection)responses are returned in a format that can be **easily executed using the Web3 library of your choice**

{% hint style="info" %}
This API is intended for public use. If you are an integrator who needs access higher rate limits, please fill out the [0x API Taker Integration Request Form.](https://www.0x.org/#contact)
{% endhint %}

![0x API's smart order routing algorithm helps users get the best price by splitting the swap across different DEXes.](<../.gitbook/assets/Screen Shot 2022-02-12 at 8.53.40 PM.png>)

## Get Started

This section contains the following docs and guides

#### API References

* [api-references](api-references/ "mention")
* [get-swap-v1-quote.md](api-references/get-swap-v1-quote.md "mention")
* [get-swap-v1-price.md](api-references/get-swap-v1-price.md "mention")
* [get-swap-v1-sources.md](api-references/get-swap-v1-sources.md "mention")

#### Advanced Topics

* [rate-limiting.md](advanced-topics/rate-limiting.md "mention")
* [how-to-set-your-token-allowances.md](advanced-topics/how-to-set-your-token-allowances.md "mention")
* [slippage-protection.md](advanced-topics/slippage-protection.md "mention")
* [price-impact-protection.md](advanced-topics/price-impact-protection.md "mention")

#### Guides

* [swap-tokens-with-0x-swap-api.md](guides/swap-tokens-with-0x-swap-api.md "mention")
* [use-0x-api-liquidity-in-your-smart-contracts.md](guides/use-0x-api-liquidity-in-your-smart-contracts.md "mention")
* [accessing-rfq-liquidity-on-0x-api.md](guides/accessing-rfq-liquidity-on-0x-api.md "mention")
* ****[**How to Build a Token Swap Dapp With 0x API**](https://docs.alchemy.com/alchemy/road-to-web3/weekly-learning-challenges/9.-how-to-build-a-token-swap-dapp-with-0x-api)****
* [troubleshooting-0x-api-swaps.md](guides/troubleshooting-0x-api-swaps.md "mention")
* [working-in-the-testnet.md](../limit-orders-advanced-traders/guides/working-in-the-testnet.md "mention")

**API FAQs**

* [faqs-and-troubleshooting.md](../developer-resources/faqs-and-troubleshooting.md "mention")
