---
description: >-
  Say goodbye to MEV attacks and say hello 👋 to the best executed price with
  Slippage Protection
---

# Slippage Protection

Topics covered on this page include:&#x20;

[#what-is-slippage-protection](slippage-protection.md#what-is-slippage-protection "mention")

[#what-is-the-best-executed-price](slippage-protection.md#what-is-the-best-executed-price "mention")

[#why-did-we-launch-slippage-protection](slippage-protection.md#why-did-we-launch-slippage-protection "mention")

[#how-does-slippage-protection-work](slippage-protection.md#how-does-slippage-protection-work "mention")

[#how-can-i-implement-slippage-protection-in-my-0x-api-quotes](slippage-protection.md#how-can-i-implement-slippage-protection-in-my-0x-api-quotes "mention")

[#read-more](slippage-protection.md#read-more "mention")

## What is Slippage Protection?

Slippage Protection is a feature of the 0x API that finds the best routes for decentralized exchange (DEX) trades while avoiding [slippage](https://help.matcha.xyz/en/articles/6304010-what-is-slippage) and MEV attacks.&#x20;

Slippage Protection incorporates slippage forecasts into 0x API’s smart order routing algorithm to deliver the optimal trade route. With Slippage Protection activated, 0x API will enable developers to surface more reliable quotes and consistently **deliver the best executed price to users**.\
\
Slippage Protection is currently supported on Ethereum for the most active trading pairs (ETH-USDC, ETH-DAI, ETH-USDT, ETH-WBTC, WETH-USDC, WETH-DAI, WETH-USDT, WETH-WBTC)

### What is the best executed price?&#x20;

The best executed price is the price users receive that’s inclusive of gas fees and slippage after taking into account all possible liquidity paths. This is an important distinction because many DeFi applications today show quoted prices which may be drastically different from the executed price.

<figure><img src="../../.gitbook/assets/Screen Shot 2022-10-04 at 10.55.30 PM.png" alt=""><figcaption><p>Without Slippage Protection, the user is routed to Liquidity Source A because it offers the best quoted price. However, if Slippage Protection is enabled, 0x API’s smart order routing instead attempts to maximize the executed price, so the user is routed to Liquidity Source B instead.</p></figcaption></figure>

## Why did we launch Slippage Protection ?

Over the past couple of years, we’ve seen significant growth in adoption of DeFi products. In 2021 alone, DEXes reported more than $1 trillion in trading volume. While this volume growth has been incredible, we’ve also seen millions of dollars lost to MEV. Since 2020, bots have extracted [over $650M](https://explore.flashbots.net/) from DEX users. These losses typically manifest as slippage, which is only visible after trade settlement (i.e. after the trade has been mined into a block). Slippage is typically left unreported to the afflicted user.&#x20;

## How does Slippage Protection work?

Slippage Protection ingests slippage measurements and their statistical properties to model and predict slippage outcomes for different liquidity sources, trading pairs, and trade sizes. These model predictions are then incorporated into 0x API’s smart order routing to identify the route(s) expected to yield the best executed price. The result is a more transparent and consistently better executed price than what users encounter today in wallets and DeFi applications.

![](<../../.gitbook/assets/Desktop - 17 (2).jpg>)

In April 2022, we published a [research piece](https://blog.0x.org/measuring-the-impact-of-hidden-dex-costs/) that shows the extent of slippage experienced by DEX users and in which circumstances slippage is more likely to occur. We leveraged this research to develop Slippage Protection and protect users against slippage. The data we collected gives us the ability to measure and predict slippage from different liquidity sources, we then incorporate those predictions into our smart order routing. Over time, Slippage Protection will continue to evolve with more data sets and the algorithm will improve.\
\
With Slippage Protection activated, orders will be routed to avoid slippage and MEV attacks, find the lowest gas fees, and deliver the best executed price. The result is a more transparent and consistently better executed price than what users encounter today in DeFi applications.\
\
For more details on how Slippage Protection works, please read[ Introducing Slippage Protection for 0x API](https://blog.0x.org/slippage-protection-for-0x-api/). \
\
Read about the performance [Post-launch of Slippage Protection](https://blog.0x.org/post-launch-of-slippageprotection-a-look-at-our-performance/).

## **How can I implement** Slippage Protection in my 0x API Quotes?&#x20;

**Slippage Protection is an auto-enabled feature of the 0x API**, and no additional action is required to enable to implement it in your API request. It is controlled by the new [`enableSlippageProtection`parameter](slippage-protection.md#request-parameter-changes).&#x20;

\
The following section includes a summary of the Request and Response changes related to Slippage Protection:

### **Changes to `/swap/quote` and `/swap/price`**

#### **Request Parameter Changes**

The `enableSlippageProtection` parameter (an Optional Boolean) has been added to the [**`/swap/quote`**](../api-references/get-swap-v1-quote.md) and [**`/swap/price`**](../api-references/get-swap-v1-price.md) endpoints.

* If it is set to `true` __ or _not set_, the quote will automatically adjust for MEV-aware slippage.
* If set to `false`, the quote returned will not be adjusted for MEV-aware slippage.

{% hint style="warning" %}
We recommended meta-aggregators and integrators who compare the quoted price with other sources to set the `enableSlippageProtection`to `false`.&#x20;
{% endhint %}

#### **Response Changes**

The `buyAmount` , `price`, and `expectedSlippage` response fields in the [**`/swap/quote`**](../api-references/get-swap-v1-quote.md) and [**`/swap/price`**](../api-references/get-swap-v1-price.md) endpoints are affected by this change

* `buyAmount` & `price`: It should be noted that when `enableSlippageProtection` is not false, the `buyAmount` & `price` returned will factor slippage in its routing.
* &#x20;`expectedSlippage`: This is the expected slippage used in routing calculations for the quote returned. It is the value of slippage that we estimate that the selected route will have:
  * It can be used by integrators to calculate the Final Expected Amount for the asset: i.e. calculated as (`buyAmount` \* `expectedSlippage` )&#x20;
  * It will only be returned when `enableSlippageProtection` is not set to `false`

## Read More

* (Blog)[ Introducing SlippageProtection for 0x API](https://blog.0x.org/slippage-protection-for-0x-api/)
* (Blog)[ Measuring the impact of hidden DEX costs](https://blog.0x.org/measuring-the-impact-of-hidden-dex-costs/)
* (Blog) [Post-launch of Slippage Protection: A look at our performance](https://blog.0x.org/post-launch-of-slippageprotection-a-look-at-our-performance/)
