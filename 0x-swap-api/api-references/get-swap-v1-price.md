# GET /swap/v1/price

`/swap/v1/price` can be used to acquire an indicative price for a transaction.

Rather than returning a transaction that can be submitted to an Ethereum node, this resource simply indicates the pricing that would be available for an analogous call to `/swap/v1/quote`.&#x20;

Intended for use with RFQ-T; see [Indicative Pricing & Firm Quotes ](../../market-makers/docs/introduction.md#indicative-pricing-and-firm-quotes)for more details.

## Request

Identical to the request schema for `/swap/v1/quote`, with a few exceptions on all supported chains:

* For `/price`, the default of `skipValidation=true` but can be overridden to `false`, whereas for `/quote`, the default of `skipValidation=false` but can be overridden to `true`
* The `intentOnFilling` field will always be undefined.

| Query Parameter            | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | Example                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `sellToken`                | The ERC20 token address or symbol of the token you want to send. Native token such as "ETH" can be provided as a valid `sellToken`. If the symbol given is not supported, try using token address instead.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | sellToken=ETH, sellToken=0x6b175474e89094c44da98b954eedeac495271d0f                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| `buyToken`                 | The ERC20 token address or symbol of the token you want to receive. Native token such as "ETH" can be provided as a valid `buyToken`. If the symbol given is not supported, try using token address instead.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | buyToken=ETH, buyToken=0x6b175474e89094c44da98b954eedeac495271d0f                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `sellAmount`               | Optional) The amount of `sellToken` (in `sellToken` base units) you want to send.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | sellAmount=100000000000                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `buyAmount`                | (Optional) The amount of `buyToken` (in `buyToken` base units) you want to receive.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | buyAmount=100000000000                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `slippagePercentage`       | <p>(Optional) The maximum acceptable slippage of the <code>buyToken</code> amount if <code>sellAmount</code> is provided; The maximum acceptable slippage of the <code>sellAmount</code> amount if <code>buyAmount</code> is provided. E.g 0.03 for 3% slippage allowed. </p><p><br>The lowest possible value that can be set for this parameter is 0; in other words, no amount of  slippage would be allowed. </p><p></p><p>If no value for this optional parameter is provided in the API request, the default slippage percentage is 1%.</p>                                                                                                                                                                                                                                                                                                            | slippagePercentage=0.03                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `gasPrice`                 | (Optional, defaults to ethgasstation "fast") The target gas price (in wei) for the swap transaction. If the price is too low to achieve the quote, an error will be returned.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | gasPrice=1000000                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `takerAddress`             | (Optional) The address which will fill the quote. When provided the gas will be estimated and returned and the entire transaction will be validated for success. If the validation fails a Revert Error will be returned in the response. The quote should be fillable if this address is provided. For example, make sure this address has enough token balance.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | takerAddress=0xa8aac589a67ecfade31efde49a062cc21d68a64e                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `excludedSources`          | (Optional) Liquidity sources (`Uniswap`, `SushiSwap`, `0x`, `Curve` etc) that will not be included in the provided quote. See [here](https://github.com/0xProject/protocol/blob/4f32f3174f25858644eae4c3de59c3a6717a757c/packages/asset-swapper/src/utils/market\_operation\_utils/types.ts#L38) for a full list of sources                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | excludedSources=Uniswap\_V3,SushiSwap,Curve                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `includedSources`          | (Optional) For now only supports `RFQT`, which should be used when the integrator only wants RFQ-T liquidity without any other DEX orders. Requires a particular agreement with the 0x integrations team. This parameter cannot be combined with `excludedSources`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | includedSources=RFQT                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `skipValidation`           | <p>(Optional) If a <code>takerAddress</code> is provided, the API can perform validations for the user. (For more details, see "<a href="../../developer-resources/faqs-and-troubleshooting.md#how-does-takeraddress-help-with-catching-issues">How does <code>takerAddress</code> help with catching issues?</a>".) When this parameter is set to <code>false</code>, that validation will be run. </p><p></p><p>Note: in the <code>/quote</code> request, validation is the default behavior (see <a href="https://docs.0x.org/market-makers/docs/introduction#quote-validation">here</a>). In <code>/price</code> requests, validation must be forced by setting  <code>skipValidation=false</code>. </p>                                                                                                                                                | skipValidation=false                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `feeRecipient`             | <p>(Optional) The ETH address that should receive affiliate fees specified with <code>buyTokenPercentageFee</code>. Can be used combination with <code>buyTokenPercentageFee</code> to set a commission/trading fee when using the API. <br><br>Learn more about how to setup a trading fee/commission fee/transaction fee <a href="https://docs.0x.org/developer-resources/faqs#i-am-building-a-dex-app-using-0x-api.-can-i-charge-my-users-a-trading-fee-commission-fee-transaction">here</a> and <a href="https://docs.0x.org/developer-resources/faqs#how-is-a-fee-returned-by-the-api-is-it-part-of-the-quoted-price-or-is-it-a-separate-parameter">here</a>. </p>                                                                                                                                                                                     | feeRecipient=0xa8aac589a67ecfade31efde49a062cc21d68a64e                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `buyTokenPercentageFee`    | <p>(Optional) The percentage (between 0 - 1.0) of the buyAmount that should be attributed to <code>feeRecipient</code> as affiliate fees. </p><p></p><p>Note that this requires that the <code>feeRecipient</code> parameter is also specified in the request.<br><br>Learn more about how to setup a trading fee/commission fee/transaction fee <a href="https://docs.0x.org/developer-resources/faqs#i-am-building-a-dex-app-using-0x-api.-can-i-charge-my-users-a-trading-fee-commission-fee-transaction">here</a> and <a href="https://docs.0x.org/developer-resources/faqs#how-is-a-fee-returned-by-the-api-is-it-part-of-the-quoted-price-or-is-it-a-separate-parameter">here</a>. </p>                                                                                                                                                               | buyTokenPercentageFee=0.1                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `affiliateAddress`         | <p>(Required) An ETH address for which to attribute the trade for tracking and analytics purposes. It can ben any valid ETH address, since it’s just for tagging purposes.<br><br>Passing this parameter with your API queries also allows us to provide you data about total volume traded in your application, pairs, MAUs, etc. in a dashboard similar to <a href="https://metabase.spaceship.0x.org/public/dashboard/e79bb86a-6777-4655-88fd-6453fdbefe0f?affiliate_address=&#x26;start_date=2022-01-01">this</a>. <br><br>Additionally, affiliateAddress will allow your project to get metrics on your user's trades via upcoming analytics tools (e.g. 0x Explorer and 0x Data API)<br><br>Note <code>affiliateAddress</code> is only for tracking trades and has no impact on affiliate fees, for affiliate fees use <code>feeRecipient</code>.</p> | <p>affiliateAddress=0xa8aac589a67ecfade31efde49a062cc21d68a64e<br><br>Here’s an example request (you just need to replace with your chosen affiliateAddress): <a href="https://api.0x.org/swap/v1/quote?sellToken=DAI&#x26;sellAmount=100000000000000000000&#x26;buyToken=WETH&#x26;feeRecipient=0x46B5BC959e8A754c0256FFF73bF34A52Ad5CdfA9&#x26;buyTokenPercentageFee=0.01&#x26;affiliateAddress=0xaae6414A06BbA56D35A80c0Ae96456280EDFd1da">https://api.0x.org/swap/v1/quote?sellToken=DAI&#x26;sellAmount=100000000000000000000&#x26;buyToken=WETH&#x26;feeRecipient=0x46B5BC959e8A754c0256FFF73bF34A52Ad5CdfA9&#x26;buyTokenPercentageFee=0.01&#x26;affiliateAddress=0xaae6414A06BbA56D35A80c0Ae96456280EDFd1da</a></p> |
| `enableSlippageProtection` | <p>(Optional) A boolean field, set to <code>true</code> or <code>false.</code><br><code></code></p><p>If <code>enableSlippageProtection</code> is either not set or set to true, the quote will be adjusted for MEV-aware slippage.<br><br>If <code>enableSlippageProtection</code> is false, the quote returned will not be adjusted for MEV-aware slippage (this is recommended for meta-aggregators &#x26; integrators who will compare the quoted price with other sources).<br><br>See affects on <code>buyAmount</code> , <code>price</code>, and <code>expectedSlippage</code> in the <a href="https://docs.0x.org/0x-api-swap/api-references/get-swap-v1-price">Response fields.</a><br><br>Read more about <a href="../advanced-topics/slippage-protection.md">Slippage Protection</a>.</p>                                                        | enableSlippageProtection=true                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |

## Response

Identical to the response schema for `/swap/v1/quote`, with a few exceptions:&#x20;

* the `orders` property will always be undefined
* `guaranteedPrice`, `to`  and `data` fields will always be undefined

| Field                  | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `price`                | <p>If <code>buyAmount</code> was specified in the request it provides the price of <code>buyToken</code> in <code>sellToken</code> and vice versa. This price does not include the <code>slippage</code> provided in the request above, and therefore represents the best possible price.<br><br>If <code>enableSlippageProtection</code> in the request was not false, the <code>buyAmount</code> &#x26; <code>price</code> responses returned will factor slippage in its routing.</p> |
| `estimatedPriceImpact` | <p>Estimated change in the price of the specified asset that would be caused by the executed swap.<br><br>Note : If we fail to estimate price change we will return <code>null</code></p>                                                                                                                                                                                                                                                                                                |
| `value`                | The amount of ether (in wei) that should be sent with the transaction. (Assuming `protocolFee` is paid in ether).                                                                                                                                                                                                                                                                                                                                                                        |
| `gasPrice`             | The gas price (in wei) that should be used to send the transaction.                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `gas`                  | The estimated gas limit that should be used to send the transaction to guarantee settlement. While a computed estimate is returned in all responses, an accurate estimate will only be returned if a `takerAddress` is included in the request.                                                                                                                                                                                                                                          |
| `estimatedGas`         | The estimate for the amount of gas that will actually be used in the transaction. Always less than `gas`.                                                                                                                                                                                                                                                                                                                                                                                |
| `protocolFee`          | The maximum amount of ether that will be paid towards the protocol fee (in wei), and what is used to compute the `value` field of the transaction.                                                                                                                                                                                                                                                                                                                                       |
| `minimumProtocolFee`   | The minimum amount of ether that will be paid towards the protocol fee (in wei) during the transaction.                                                                                                                                                                                                                                                                                                                                                                                  |
| `buyTokenAddress`      | The ERC20 token address of the token you want to receive in quote.                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `buyAmount`            | <p>The amount of <code>buyToken</code> (in <code>buyToken</code> units) that would be bought in this swap. Certain on-chain sources do not allow specifying <code>buyAmount</code>, when using <code>buyAmount</code> these sources are excluded.<br><br>If <code>enableSlippageProtection</code> in the request was not false, the <code>buyAmount</code> &#x26; <code>price</code> responses returned will factor slippage in its routing.</p>                                         |
| `sellTokenAddress`     | The ERC20 token address of the token you want to sell with quote.                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `sellAmount`           | The amount of `sellToken` (in `sellToken` units) that would be sold in this swap. Specifying `sellAmount` is the recommended way to interact with 0xAPI as it covers all on-chain sources.                                                                                                                                                                                                                                                                                               |
| `sources`              | The percentage distribution of `buyAmount` or `sellAmount` split between each liquidity source. Ex: \[{ name: '0x', proportion: "0.8" }, { name: 'Kyber', proportion: "0.2"}, ...]                                                                                                                                                                                                                                                                                                       |
| `allowanceTarget`      | The target contract address for which the user needs to have an allowance in order to be able to complete the swap. For swaps with "ETH" as `sellToken`, wrapping "ETH" to "WETH" or unwrapping "WETH" to "ETH" no allowance is needed, a null address of `0x0000000000000000000000000000000000000000` is then returned instead.                                                                                                                                                         |
| `sellTokenToEthRate`   | The rate between ETH and `sellToken`                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `buyTokenToEthRate`    | The rate between ETH and `buyToken`                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `expectedSlippage`     | <p>This is the expected slippage used in routing calculations for the quote returned. It is the value of slippage that we estimate that the selected route will have: </p><ul><li>It can be used by integrators to calculate the Final Expected Amount for the asset: i.e. calculated as (<code>buyAmount</code> * <code>expectedSlippage</code> ) </li><li>It will only be returned when <code>enableSlippageProtection</code> is not set to <code>false</code></li></ul>               |

## Example

### **Get the price available for selling 1 ETH for DAI**

**Request**

GET

```
https://api.0x.org/swap/v1/price?sellToken=ETH&buyToken=DAI&sellAmount=1000000000000000000
```

#### **Response**

```
{
    "chainId": 1,
    "price": "2690.904091272912844972",
    "estimatedPriceImpact": "0",
    "value": "1000000000000000000",
    "gasPrice": "140000000000",
    "gas": "151000",
    "estimatedGas": "151000",
    "protocolFee": "0",
    "minimumProtocolFee": "0",
    "buyTokenAddress": "0x6b175474e89094c44da98b954eedeac495271d0f",
    "buyAmount": "2690904091272912844972",
    "sellTokenAddress": "0xeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee",
    "sellAmount": "1000000000000000000",
    "sources": [
        {
            "name": "0x",
            "proportion": "0"
        },
        {
            "name": "Uniswap",
            "proportion": "0"
        },
        {
            "name": "Uniswap_V2",
            "proportion": "0"
        },
        {
            "name": "Eth2Dai",
            "proportion": "0"
        },
        {
            "name": "Kyber",
            "proportion": "0"
        },
        {
            "name": "Curve",
            "proportion": "0"
        },
        {
            "name": "Balancer",
            "proportion": "0"
        },
        {
            "name": "Balancer_V2",
            "proportion": "0"
        },
        {
            "name": "Bancor",
            "proportion": "0"
        },
        {
            "name": "mStable",
            "proportion": "0"
        },
        {
            "name": "Mooniswap",
            "proportion": "0"
        },
        {
            "name": "Swerve",
            "proportion": "0"
        },
        {
            "name": "SnowSwap",
            "proportion": "0"
        },
        {
            "name": "SushiSwap",
            "proportion": "0"
        },
        {
            "name": "Shell",
            "proportion": "0"
        },
        {
            "name": "MultiHop",
            "proportion": "0"
        },
        {
            "name": "DODO",
            "proportion": "0"
        },
        {
            "name": "DODO_V2",
            "proportion": "0"
        },
        {
            "name": "CREAM",
            "proportion": "0"
        },
        {
            "name": "LiquidityProvider",
            "proportion": "0"
        },
        {
            "name": "CryptoCom",
            "proportion": "0"
        },
        {
            "name": "Linkswap",
            "proportion": "0"
        },
        {
            "name": "Lido",
            "proportion": "0"
        },
        {
            "name": "MakerPsm",
            "proportion": "0"
        },
        {
            "name": "KyberDMM",
            "proportion": "0"
        },
        {
            "name": "Smoothy",
            "proportion": "0"
        },
        {
            "name": "Component",
            "proportion": "0"
        },
        {
            "name": "Saddle",
            "proportion": "0"
        },
        {
            "name": "xSigma",
            "proportion": "0"
        },
        {
            "name": "Uniswap_V3",
            "proportion": "1"
        },
        {
            "name": "Curve_V2",
            "proportion": "0"
        },
        {
            "name": "ShibaSwap",
            "proportion": "0"
        },
        {
            "name": "Synapse",
            "proportion": "0"
        }
    ],
    "allowanceTarget": "0x0000000000000000000000000000000000000000",
    "sellTokenToEthRate": "1",
    "buyTokenToEthRate": "2684.34399396039791937"
}
```
