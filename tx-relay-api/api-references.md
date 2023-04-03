# API References

### Overview

There are currently four endpoints involved in a tx relay transaction:

* `/tx-relay/v1/swap/price`
* `/tx-relay/v1/swap/quote`
* `/tx-relay/v1/swap/submit`
* `/tx-relay/v1/swap/status/:trade-hash`

## `/tx-relay/v1/swap/price`

Indicative pricing can be obtained by submitting a GET request to `/tx-relay/v1/swap/price`. This endpoint should be used when the end-user does not necessarily want to trade, as it can handle more traffic than the other endpoints.

#### Example Request

⚠️ An API key should always be specified when requesting all possible sources of liquidity. API keys are specified via a header parameter called `0x-api-key`. Chain ids are specified via a header parameter called `0x-chain-id` which currently supports `1` (Ethereum) and `137` (Polygon).

⚠️ `sellToken`, `buyToken`, `takerAddres` and one of \[`sellAmount` ,`buyAmount` ] must be present

```bash
curl '<https://api.0x.org/tx-relay/v1/swap/price?buyToken=0x0d500B1d8E8eF31E21C99d1Db9A6444d3ADf1270&sellAmount=100000000&sellToken=0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174&takerAddress=0x9016Cc2122b52fF5d9937c0c1422B78d7e81CeEa&priceImpactProtectionPercentage=0.95&feeType=volume&feeSellTokenPercentage=0.1&feeRecipient=0x70A9f34F9b34C64957b9c401A97BfeD35b95049e>' \\
--header '0x-api-key: <API_KEY>' --header '0x-chain-id: 137'
```

* `buyToken`: The contract address of the token being bought.
* `sellToken`: The contract address of token being sold.
* `buyAmount`: The amount of `buyToken` to buy. Can only be present if `sellAmount` is not present.
* `sellAmount`: The amount of `sellToken` to sell. Can only be present if `buyAmount` is not present.
* `takerAddress`: The address of the taker.
* `acceptedTypes`: \[optional] Comma delimited string of the types of order the caller is willing to receive
  * Currently, _**only**_ `metatransaction` is supported and allowed. In the near future, `otc` and `metatransaction_v2` will be added.
  * This is useful if the caller only wants to receive types the caller specifies. _**If not provided, it means the caller accepts any types**_.
* `slippagePercentage`: \[optional] The maximum amount of slippage acceptable to the user; any slippage beyond that specified will cause the transaction to revert on chain. **Default is 1% and minimal value allowed is 0.1%.** The value of the field is on scale of 1. For example, setting `slippagePercentage` to set to `0.01` means `1%` slippage allowed.
* `priceImpactProtectionPercentage`: \[optional] The maximum amount of price impact acceptable to the user; any price impact beyond that specified will cause the endpoint to return error. The value of the field is on scale of 1. For example, setting `priceImpactProtectionPercentage` to set to `0.01` means `1%` price impact allowed
  * This is an _**opt-in**_ feature, the default value of _**1.0**_ will disable the feature. When it is set to 1.0 (100%) it means that _**every transaction is allowed to pass**_.
  * Price impact calculation includes all fees taken.
  * Read more about price impact at [0x documentation](https://docs.0x.org/0x-swap-api/advanced-topics/price-impact-protection).
* `feeType`: \[optional] The type of integrator fee to charge. The current allowed value is `volume`.
* `feeRecipient`: \[optional] The address the integrator fee would be transferred to. This is the address you’d like to receive the fee. This must be present if `feeType` is provided
* `feeSellTokenPercentage` : \[optional] If `feeType` is `volume`, then `feeSellTokenPercentage` must be provided. `feeSellTokenPercentage` is the percentage (on scale of 1) of `sellToken` integrator charges as fee. For example, setting it to `0.01` means `1%` of the `sellToken` would be charged as fee for the integrator.

#### Example Response

**Liquidity Unavailable Response**

```json
{
    "liquidityAvailable": false,
}
```

**Response if liquidity is available**

```json
{
    "liquidityAvailable": true,
    "price": "391.1643362",
	  "estimatedPriceImpact": "5",
    "buyTokenAddress": "0x0d500B1d8E8eF31E21C99d1Db9A6444d3ADf1270",
    "buyAmount": "391164336200000000000",
    "sellTokenAddress": "0x2791bca1f2de4661ed88a30c99a7a9449aa84174",
    "sellAmount": "100000000",
    "allowanceTarget": "0xdef1c0ded9bec7f1a1670819833240f027b25eff",
    "sources": [{"name":"0x","proportion":"0"},{"name":"Uniswap","proportion":"0"},{"name":"Uniswap_V2","proportion":"1"}],
		"fees": {
			"integratorFee": {
				"feeType": "volume",
			  "feeToken": "0x2791bca1f2de4661ed88a30c99a7a9449aa84174",
				"feeAmount": "10000"
      },
			"zeroExFee": {
				"feeType": "volume",
				"feeToken": "0x2791bca1f2de4661ed88a30c99a7a9449aa84174",
				"feeAmount": "100"
			},
			"gasFee": {
				"feeType": "gas",
				"feeToken": "0x2791bca1f2de4661ed88a30c99a7a9449aa84174",
				"feeAmount": "1"
			}
		}
}
```

* `liquidityAvailable`: Validates that liquidity is available. This would always been present.
* The rest of the fields would only be present if `liquidityAvailable` is `true`
* `price`: If `buyAmount` was specified in the request, this parameter provide the price of `buyToken`, denominated in `sellToken`, or vice-versa.
* `estimatedPriceImpact`: The estimated change in the price of the specified asset that would be caused by the executed swap due to [price impact](https://docs.0x.org/0x-swap-api/advanced-topics/price-impact-protection). Note that if the API is not able to estimate price change, the field will be `null`.
* `buyTokenAddress`: The ERC20 token address of the token you want to receive in quote.
* `buyAmount`: The amount of `buyToken` to buy.
* `sellTokenAddress`: The ERC20 token address of the token you want to sell with quote.
* `sellAmount`: The amount of `sellToken` to sell.
* `allowanceTarget`: The target contract address for which the user needs to have an allowance in order to be able to complete the swap.
* `sources`: The underlying sources for the liquidity. The format will be:
  * `[{ name: string; proportion: string }]`
  * An example: `[{"name":"0x","proportion":"0"},{"name":"Uniswap","proportion":"0"},{"name":"Uniswap_V2","proportion":"1"},...]`
* `fees`: \[optional] Fees that would be charged. It can optionally contain `integratorFee`, `zeroExFee` and `gasFee`:
  * `integratorFee`: Integrator fee (in amount of `sellToken`) would be provided if `feeType` and the corresponding query params are provided in the request:
    * `feeType`: The type of the `integrator` fee. This is always the same as the `feeType` in the request. It can only be `volume` currently.
    * `feeToken`: The ERC20 token address to charge fee. This is always be the same as `sellToken` in the request.
    * `feeAmount`: The amount of `feeToken` to be charged as integrator fee. Note this amount includes the possible 0x fee if 0x charges integrator fee described below.
  * `zeroExFee`: fee that 0x charges (in amount of `sellToken`):
    * `feeType`: `volume` | `integrator_share`
    * `feeToken`: The ERC20 token address to charge fee. This is always be the same as `sellToken` in the request.
    * `feeAmount`: The amount of `feeToken` to be charged as `0x` fee.
  * `gasFee`: gas fee (in amount of `sellToken`) to compensate for the transaction submission performed by our workers:
    * `feeType`: The value is always `gas`.
    * `feeToken`: The ERC20 token address to charge gas fee. This is always be the same as `sellToken` in the request.
    * `feeAmount`: The amount of `feeToken` to be charged as gas fee.

## `/tx-relay/v1/swap/quote`

Fillable quotes are obtained by submitting a GET request to `/tx-relay/v1/swap/quote`

#### **Example Request**

The request parameters are the same as `[/price](<https://www.notion.so/0x-Tx-Relay-Partner-Hub-2f0074ebe816409b958ab013da13c570>)` except with an extra field `checkApproval`:

```bash
curl '<https://api.0x.org/tx-relay/v1/swap/quote?buyToken=0x0d500B1d8E8eF31E21C99d1Db9A6444d3ADf1270&sellAmount=100000000&sellToken=0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174&takerAddress=0x9016Cc2122b52fF5d9937c0c1422B78d7e81CeEa&priceImpactProtectionPercentage=0.95&feeType=volume&feeSellTokenPercentage=0.1&feeRecipient=0x70A9f34F9b34C64957b9c401A97BfeD35b95049e>' \\
--header '0x-api-key: <API_KEY>' --header '0x-chain-id: 137'
```

* `checkApproval`: \[optional] Whether to check for approval and potentially utilizes gasless approval feature. Allowed values `true` / `false`. Default to `false` if not provided. Setting it to `true` would require more processing and computation on our end so it would have worse performance.

#### Example Response

The response will initially be of type `metatransaction`. There are two new types `metatransaction_v2` and `otc` currently under development and will be available in the near future.

Thus, please don’t assume particular shapes of `trade.eip712.types`, `trade.eip712.domain`, `trade.eip712.primaryType` and `trade.eip712.message` as they will change based on types returned, namely `metatransaction` , `metatransaction_v2` and `otc`. More types might also be added in the future.

Similarly for `approval.eip712.types`, `approval.eip712.domain`, `approval.eip712.primaryType` and `approval.eip712.message` as there are different types of gasless approval standards.

**Liquidity Unavailable Response**

```json
{
    "liquidityAvailable": false,
}
```

**Example Meta-Transaction Response**

```json
{
  "liquidityAvailable": true,
  "price": "1800.054805",
  "estimatedPriceImpact": "5",
  "buyAmount": "1800054805473",
  "buyTokenAddress": "0x0d500b1d8e8ef31e21c99d1db9a6444d3adf1270",
  "sellAmount": "100000000",
  "sellTokenAddress": "0x2791bca1f2de4661ed88a30c99a7a9449aa84174",
	"allowanceTarget": "0xdef1c0ded9bec7f1a1670819833240f027b25eff",
  "sources": [{"name":"0x","proportion":"0"},{"name":"Uniswap","proportion":"0"},{"name":"Uniswap_V2","proportion":"1"}],
	"fees": {
		// same as the response in /price. Redacted here
	},
  "trade": {
    "type": "metatransaction",
    "hash": "0xde5a11983edd012047dd3107532f007a73ae488bfb354f35b8a40580e2a775a1",
    "eip712": {
      "types": {
	      "EIP712Domain": [
			    {
	          "name": "name",
            "type": "string"
          },
          {
            "name": "version",
            "type": "string"
          },
          {
            "name": "chainId",
            "type": "uint256"
          },
          {
            "name": "verifyingContract",
            "type": "address"
          }
        ],
			  "MetaTransactionData": [
			    {
			      "type": "address",
			      "name": "signer"
			    },
			    {
			      "type": "address",
			      "name": "sender"
			    },
			    {
			      "type": "uint256",
			      "name": "minGasPrice"
			    },
			    {
			      "type": "uint256",
			      "name": "maxGasPrice"
			    },
			    {
			      "type": "uint256",
			      "name": "expirationTimeSeconds"
			    },
			    {
			      "type": "uint256",
			      "name": "salt"
			    },
			    {
			      "type": "bytes",
			      "name": "callData"
			    },
			    {
			      "type": "uint256",
			      "name": "value"
			    },
			    {
			      "type": "address",
			      "name": "feeToken"
			    },
			    {
			      "type": "uint256",
			      "name": "feeAmount"
			    }
			  ]
      },
      "domain": {
        "name": "ZeroEx",
        "version": "1.0.0",
        "chainId": 1,
        "verifyingContract": "0xdef1c0ded9bec7f1a1670819833240f027b25eff"
      },
      "primaryType": "MetaTransactionData",
      "message": {
        "signer": "0x9016cc2122b52ff5d9937c0c1422b78d7e81ceea",
        "sender": "0x0000000000000000000000000000000000000000",
		    "minGasPrice": "1",
	      "maxGasPrice": "4294967296",
        "expirationTimeSeconds": "9990868679",
        "salt": "32606650794224190000000000000000000000000000000000000000000000000000000000000",
        "callData": "0x415565b00000000000000000000000007ceb23fd6bc0.....",
				"value": "0",
	      "feeToken": "0x0000000000000000000000000000000000000000",
        "feeAmount": "0"
      }
    }
	},
  "approval": {
    "isRequired": true,
    "isGaslessAvailable": true,
	  "type": "permit",
		"hash": "0x9d5435a70c77ffc36b1dd5d2f05ce5edcb1d0f52e2e134c3ad957b421deae194",
    "eip712": {
      "types": {
        "EIP712Domain": [
          {
            "name": "name",
            "type": "string"
          },
          {
            "name": "version",
            "type": "string"
          },
					{
						"name": "chainId",
						"type": "uint256",
					},
          {
            "name": "verifyingContract",
            "type": "address"
          }
        ],
        "Permit": [
          {
            "name": "owner",
            "type": "address"
          },
          {
            "name": "spender",
            "type": "address"
          },
          {
            "name": "value",
            "type": "uint256"
          },
          {
            "name": "nonce",
            "type": "uint256"
          },
          {
            "name": "deadline",
            "type": "uint256"
          }
        ]
      },
      "primaryType": "Permit",
      "domain": {
	      "name": "USD Coin",
        "version": "2",
	      "chainId": 1,
        "verifyingContract": "0xa0b86991c6218b36c1d19d4a2e9eb0ce3606eb48"
      },
      "message": {
        "owner": "0x9016cc2122b52ff5d9937c0c1422b78d7e81ceea",
        "spender": "0xdef1c0ded9bec7f1a1670819833240f027b25eff",
        "value": "115792089237316195423570985008687907853269984665640564039457584007913129639935",
        "nonce": 0,
        "deadline": "1660163988"
      }
    }
  }
}
```

**Example Meta-Transaction V2 Response**

For meta-transaction v2 response, all fields are the same as [the meta-transaction response](https://www.notion.so/0x-Tx-Relay-Partner-Hub-2f0074ebe816409b958ab013da13c570) except for `trade` field. More specifically, the following fields are different:

* `trade.eip712.types`: New types`MetaTransactionDataV2` and `MetaTransactionFeeData` instead of `MetaTransactionData`
* `trade.eip712.primaryType`: `MetaTransactionDataV2` instead of `MetaTransactionData`
* `trade.eip712.message`

```json
{
  ...
	// All fields other than `trade` are the same as meta-transaction response
  "trade": {
    "type": "metatransaction_v2",
    "hash": "0xde5a11983edd012047dd3107532f007a73ae488bfb354f35b8a40580e2a775a1",
    "eip712": {
      "types": {
	      "EIP712Domain": [
			    {
	          "name": "name",
            "type": "string"
          },
          {
            "name": "version",
            "type": "string"
          },
          {
            "name": "chainId",
            "type": "uint256"
          },
          {
            "name": "verifyingContract",
            "type": "address"
          }
        ],
				// different type compared with meta-transaction response
        "MetaTransactionDataV2": [
	        {
            "type": "address",
            "name": "signer"
          },
          { 
            "type": "address",
            "name": "sender"
          },
          { 
            "type": "uint256",
            "name": "expirationTimeSeconds"
          },
          { 
            "type": "uint256",
            "name": "salt"
          },
          { 
            "type": "bytes",
            "name": "callData"
          },
          { 
            "type": "address",
            "name": "feeToken"
          },
          {
            "type": "MetaTransactionFeeData[]",
            "name": "fees"
          }
	      ],
				// different type compared with meta-transaction response
	      "MetaTransactionFeeData": [
	        {
            "type": "address",
            "name": "recipient"
	        },
	        {
            "type": "uint256",
            "name": "amount"
	        }
        ]
      },
      "domain": {
        "name": "ZeroEx",
        "version": "1.0.0",
        "chainId": 1,
        "verifyingContract": "0xdef1c0ded9bec7f1a1670819833240f027b25eff"
      },
			// different primary type compared with meta-transaction response
      "primaryType": "MetaTransactionDataV2",
			// different message with meta-transaction response
      "message": {
        "signer": "0x9016cc2122b52ff5d9937c0c1422b78d7e81ceea",
        "sender": "0x0000000000000000000000000000000000000000",
        "expirationTimeSeconds": "9990868679",
        "salt": "32606650794224190000000000000000000000000000000000000000000000000000000000000",
        "callData": "0x415565b00000000000000000000000007ceb23fd6bc0.....",
	      "feeToken": "0x7ceb23fd6bc0add59e62ac25578270cff1b9f619",
        "fees": [
            {
                "recipient": "0x70a9f34f9b34c64957b9c401a97bfed35b95049e",
                "amount": "10"
            },
            {
                "recipient": "0x5fd625230def5303c82f0d1d491041e042f2ad22",
                "amount": "15"
            }
        ]
      }
    }
	},
  ...
}
```

**Example OTC Response**

Similarly, for otc response, all fields are the same as [the meta-transaction response](https://www.notion.so/0x-Tx-Relay-Partner-Hub-2f0074ebe816409b958ab013da13c570) except for `trade` field. More specifically, the following fields are different:

* `trade.eip712.types`: New type `OtcOrder` instead of `MetaTransactionData`
* `trade.eip712.primaryType`: `MetaTransactionDataV2` instead of `MetaTransactionData`
* `trade.eip712.message`

```json
{
  ...
	// All fields other than `trade` are the same as meta-transaction response
  "trade": {
    "type": "otc",
    "hash": "0x12345...",
    "eip712": {
      "types": {
	      "EIP712Domain": [
			    {
	          "name": "name",
            "type": "string"
          },
          {
            "name": "version",
            "type": "string"
          },
          {
            "name": "chainId",
            "type": "uint256"
          },
          {
            "name": "verifyingContract",
            "type": "address"
          }
        ],
				// different type compared with meta-transaction response
        "OtcOrder": [
	        {
            "type": "address",
            "name": "makerToken"
          },
          { 
            "type": "address",
            "name": "takerToken"
          },
          { 
            "type": "uint128",
            "name": "makerAmount"
          },
          { 
            "type": "uint128",
            "name": "takerAmount"
          },
          { 
            "type": "address",
            "name": "maker"
          },
          { 
            "type": "address",
            "name": "taker"
          },
                    { 
            "type": "address",
            "name": "txOrigin"
          },
          { 
            "type": "uint256",
            "name": "expiryAndNonce"
          }
	      ]
      },
      "domain": {
        "name": "ZeroEx",
        "version": "1.0.0",
        "chainId": 1,
        "verifyingContract": "0xdef1c0ded9bec7f1a1670819833240f027b25eff"
      },
			// different primary type compared with meta-transaction response
      "primaryType": "OtcOrder",
			// different message with meta-transaction response
      "message": {
        "makerToken": "0x2791bca1f2de4661ed88a30c99a7a9449aa84174",
        "takerToken": "0x0d500b1d8e8ef31e21c99d1db9a6444d3adf1270",
        "makerAmount": "110969436",
        "takerAmount": "100000000000000000000",
        "maker": "0x876c...",
	      "taker": "0x4ea7...",
				"txOrigin": "0xc65...",
        "expiryAndNonce": "1054164..."
      }
    }
	},
  ...
}
```

* `liquidityAvailable`: Used to validate that liquidity is available from a given source. This would always be present.
* The rest of the fields would only be present if `liquidityAvailable` is `true`. Fields that have been explained previously in `[/price](<https://www.notion.so/0x-Tx-Relay-Partner-Hub-2f0074ebe816409b958ab013da13c570>)` are not included again here.
* `trade`: This is the “trade” object which contains the necessary information to process a trade
  * `type`: `metatransaction` | `metatransaction_v2` | `otc`
  * `hash`: The hash for the trade according to EIP-712 ([https://eips.ethereum.org/EIPS/eip-712](https://eips.ethereum.org/EIPS/eip-712)). Note that if you compute the hash from `eip712` field, it should match the value of this field.
  * `eip712`: Necessary data for EIP-712 ([https://eips.ethereum.org/EIPS/eip-712](https://eips.ethereum.org/EIPS/eip-712)).
    * Note: Please don’t assume particular shapes of `trade.eip712.types`, `trade.eip712.domain`, `trade.eip712.primaryType` and `trade.eip712.message` as they will change based on the `type` field and we would add more types in the future.
* `approval`: This is the “approval” object which contains the necessary information to process a gasless approval, if requested via `checkApproval` and is available.
  * `type`: `permit` | `executeMetaTransaction::approve`
  * `hash`: The hash for the approval according to EIP-712 ([https://eips.ethereum.org/EIPS/eip-712](https://eips.ethereum.org/EIPS/eip-712)). Note that if you compute the hash from `eip712` field, it should match the value of this field.
  * `eip712`: Necessary data for EIP-712 ([https://eips.ethereum.org/EIPS/eip-712](https://eips.ethereum.org/EIPS/eip-712)).
    * Note: Please don’t assume particular shapes of `approval.eip712.types`, `approval.eip712.domain`, `approval.eip712.primaryType` and `approval.eip712.message` as they will change based on the `type` field.

## `/tx-relay/v1/swap/submit`

If your user accepts the quote and wants to trade, you do a `POST` to `/tx-relay/v1/swap/submit`.

### Gasless Approvals

If a token supports gasless approvals, a meta-transaction may be submitted along with an approval object. You will be able to tell if the sell token is supported by gasless approvals by checking the response of `/quote` and looking for

```tsx
const { approval } = response;
approval.isRequired // whether an approval is required for the trade
approval.isGaslessAvailable // whether gasless approval is available for the sell token
```

To take advantage of gasless approvals, you must also have your user sign the `approval.eip712`

object returned at the time of the `/quote`. You can pass the `approval.eip712` object to `eth_signTypedData_v4` (see [MetaMask docs](https://docs.metamask.io/guide/signing-data.html#sign-typed-data-v4)) or function of your choice as the “params” .

Keep in mind that the `domain` field of this object must follow a strict ordering as specified in EIP-712:

* `name` , `version`, `chainId`, `verifyingContract`, `salt`
  * Contracts may not utilize all of these fields (i.e. one or more may be missing), but if they are present, they must be in this order
* Any other field must follow in alphabetical order

The `approval.eip712` object will present the correct ordering, but make sure that this ordering is preserved when obtaining the signature.

If you choose to compute the approval hash from `approval.eip712`, it should match `approval.hash` field.

### Meta-transaction Liquidity

Similar to gasless approval, to submit an meta-transaction trade, you must have your user sign `trade.eip712` object returned at the time of the `/quote`. All the instructions & caveats are the same as previous section.

#### **Example Request**

```bash
curl -X POST '<https://api.0x.org/tx-relay/v1/swap/submit>' --header '0x-api-key: <API_KEY>' --header '0x-chain-id: 137' --data '{
 "trade": {
   "type": "metatransaction_v2", // this is trade.type from the /quote endpoint
   "eip712": { /* this is trade.eip712 from the /quote endpoint */ },
   "signature": {
     "v": 27,
     "r": "0xeaac9ddbbf9b251a384eb4a545a2800a6d7aca4ceb5e5a3154ddd0d2923533d2",
     "s": "0x601deadfd33bd8b0ad35468613eabcac0a250a7a32035e261a13e2dcbc462e49",
     "signatureType": 2
    }
  },
 "approval":{
     "type": "permit", // this is approval.type from the /quote endpoint
     "eip712": {/* this is approval.eip712 from the /quote endpoint */},
     "signature": {
       "v": 28,
       "r": "0x55c26901285d5cb4d9d1ebb2828122fd6c334b343901944d34a810b3d2d79773",
       "s": "0x20854d829e3118c3f780228ca83fac6154060328a90d2a21267c9f7d1ae9e53b",
       "signatureType": 2
    }
  }
}'
```

#### **Example Response**

```json
{
  "type": "metatransaction_v2",
  "tradeHash": "0xde5a11983edd012047dd3107532f007a73ae488bfb354f35b8a40580e2a775a1",
}
```

More information on signing 0x orders is available [here](https://docs.0x.org/market-makers/guides/signing-0x-orders).

## **`/tx-relay/v1/swap/status/:trade-hash`**

To check the status of a trade, you submit a GET request to `/tx-relay/v1/swap/status/:trade-hash`.

#### Example **Request**

```bash
curl '<https://api.0x.org/tx-relay/v1/swap/status/0xd114c77249bb3a137634afeba1ea1c8d6080c687c1a00cc4137fd9cb905a5fb6>' \\
--header '0x-api-key: <API_KEY>' --header '0x-chain-id: 137'
```

**Shape of Response:**

```json
{
    "transactions": { hash: string; timestamp: number /* unix ms */ }[]; // trade transactions
    "approvalTransactions": { hash: string; timestamp: number /* unix ms */ }[]; // approval transactions; the field will not be present if it's not a gasless approval trade
    // For pending, expect no transactions.
    // For successful transactions (i.e. "succeeded"/"confirmed), expect just the mined transaction.
    // For failed transactions, there may be 0 (failed before submission) to multiple transactions (transaction reverted).
    // For submitted transactions, there may be multiple transactions, but only one will ultimately get mined
} & ({ status: 'pending' | 'submitted' | 'succeeded' | 'confirmed' } |
     { status: 'failed'; reason: JobFailureReason });
     // When status field is 'failed', there will be a reason field to describe the error reason
```

**Current set of failure reasons:**

```tsx
enum JobFailureReason {
    // Transaction simulation failed so no transaction is submitted onchain.
    // Our system simulate the transaction before submitting onchain.
    TransactionSimulationFailed = 'transaction_simulation_failed';
    // The order expired
    OrderExpired = 'order_expired';
    // Last look declined by the market maker
    LastLookDeclined = 'last_look_declined';
    // Transaction(s) submitted onchain but reverted
    TransactionReverted = 'transaction_reverted';
    // Error getting market signature / signature is not valid; this is NOT last look decline
    MarketMakerSignatureError = 'market_maker_sigature_error';
    // Fallback error reason
    InternalError = 'internal_error';
}
```

#### **Example Response**

```jsx
// confirmed
{
    "status": 'confirmed',
    "transactions": [{
        "hash": "0x...",
        "timestamp": 1624290253193
    }],
    "approvalTransactions": [{
       "hash": "0x...",
       "timestamp": 1624290253183
    }]
}

// failed due to transaction reverted
{
    "status": "failed",
    "transactions": [{
        "hash": "0x...",
        "timestamp": 1624290253193
    }],
    "reason": "transaction_reverted"
}
```

**Note**

`JobFailureReason.InternalError` is used as the fallback error if the error reason is not one of the errors listed in `JobFailureReason`.

In the future, 0x might add more entries in `JobFailureReason`to provide more information about the failure. Thus, if you were to switch the `reason` field, it’s recommended to use:

```tsx
switch (reason) {
    case 'transaction_simulation_failed':
        ...
    case 'order_expired':
        ...
    default:
        // case for internal_error
}
```

instead of:

```tsx
switch (reason) {
    case 'transaction_simulation_failed':
        ...
    case 'order_expired':
        ...
    case 'internal_error':
        ...
    default:
        throw
}
```

to avoid unintended errors.

## 0x Protocol

Once signed orders hit the blockchain, they are settled by 0x Protocol smart contracts. For meta-transaction orders, they are settled by the contract `MetaTransactionsFeature` and filled by the `execteMetaTransaction` function, available [here](https://github.com/0xProject/protocol/blob/main/contracts/zero-ex/contracts/src/features/MetaTransactionsFeature.sol).

## Technical Appendix

#### Presenting EIP-712 Signatures for `signTypedData`

If you are integrating with Metamask or another user facing wallet that shows the users the details of what they are signing, then you will most likely want to use the EIP-712 signing strategy. In order to do so, you will need the following:

* `domain`
* `types`
* `primaryType`
* `message`

The message will be `MetaTransactionData` (in the future `MetaTransactionDataV2`) that is returned at the time of `/quote`. However, you will also need the other fields.

The Domain will change per chain, but the `name` and `version` fields are consistent. Example:

```jsx
const domain = {
    "chainId": 1,
    "verifyingContract": "0xdef1c0ded9bec7f1a1670819833240f027b25eff",
    "name": "ZeroEx",
    "version": "1.0.0"
};
```

For `types` and `primaryTypes`, it will depend on the message format.

*   For `MetaTransactionData` (click the caret to expand)

    ```jsx
    const primaryType = "MetaTransactionData";
    const types = {
      "EIP712Domain": [
        {
          "name": "name",
          "type": "string"
        },
        {
          "name": "version",
          "type": "string"
        },
        {
          "name": "chainId",
          "type": "uint256"
        },
        {
          "name": "verifyingContract",
          "type": "address"
        }
      ],
      "MetaTransactionData": [
        {
          "type": "address",
          "name": "signer"
        },
        {
          "type": "address",
          "name": "sender"
        },
        {
          "type": "uint256",
          "name": "minGasPrice"
        },
        {
          "type": "uint256",
          "name": "maxGasPrice"
        },
        {
          "type": "uint256",
          "name": "expirationTimeSeconds"
        },
        {
          "type": "uint256",
          "name": "salt"
        },
        {
          "type": "bytes",
          "name": "callData"
        },
        {
          "type": "uint256",
          "name": "value"
        },
        {
          "type": "address",
          "name": "feeToken"
        },
        {
          "type": "uint256",
          "name": "feeAmount"
        }
      ]
    };
    ```
*   For `MetaTransactionDataV2` (currently in development; click the caret to expand)

    ```jsx
    const primaryType = "MetaTransactionDataV2";
    const types = {
        "EIP712Domain": [
    	    {
            "name": "name",
            "type": "string"
          },
          {
            "name": "version",
            "type": "string"
          },
          {
            "name": "chainId",
            "type": "uint256"
          },
          {
            "name": "verifyingContract",
            "type": "address"
          }
        ],
        "MetaTransactionDataV2": [
          {
            "type": "address",
            "name": "signer"
          },
          { 
            "type": "address",
            "name": "sender"
          },
          { 
            "type": "uint256",
            "name": "expirationTimeSeconds"
          },
          { 
            "type": "uint256",
            "name": "salt"
          },
          { 
            "type": "bytes",
            "name": "callData"
          },
          { 
            "type": "address",
            "name": "feeToken"
          },
          {
            "type": "MetaTransactionFeeData[]",
            "name": "fees"
          }
        ],
        "MetaTransactionFeeData": [
          {
            "type": "address",
            "name": "recipient"
          },
          {
            "type": "uint256",
            "name": "amount"
          }
        ]
      };
    ```
*   For `OtcOrder` (currently in development; click the caret to expand)

    ```jsx
    const primaryType = "OtcOrder";
    const types = {
        "EIP712Domain": [
    	    {
            "name": "name",
            "type": "string"
          },
          {
            "name": "version",
            "type": "string"
          },
          {
            "name": "chainId",
            "type": "uint256"
          },
          {
            "name": "verifyingContract",
            "type": "address"
          }
        ],
        "OtcOrder": [
          {
            "type": "address",
            "name": "makerToken"
          },
          { 
            "type": "address",
            "name": "takerToken"
          },
          { 
            "type": "uint128",
            "name": "makerAmount"
          },
          { 
            "type": "uint128",
            "name": "takerAmount"
          },
          { 
            "type": "address",
            "name": "maker"
          },
          { 
            "type": "address",
            "name": "taker"
          },
                    { 
            "type": "address",
            "name": "txOrigin"
          },
          { 
            "type": "uint256",
            "name": "expiryAndNonce"
          }
        ]
    };
    ```

#### Computing a trade hash

You could / should verify that the hash we provide in our request matches the meta-transaction provided.

For the `trade.hash` field:

* verify that the `meta-transaction` hashes to the `trade.hash`
* `getMetaTransactionHash`[https://github.com/0xProject/protocol/blob/development/contracts/zero-ex/contracts/src/features/MetaTransactionsFeature.sol#L204](https://github.com/0xProject/protocol/blob/development/contracts/zero-ex/contracts/src/features/MetaTransactionsFeature.sol#L204)

If you need to call these functions on the 0x smart contracts to validate your code, you may need:

* The ABI:[https://github.com/0xProject/protocol/blob/d738eede0e15d4120b18bb3f88b8aba986a3f774/packages/contract-artifacts/artifacts/IZeroEx.json](https://github.com/0xProject/protocol/blob/d738eede0e15d4120b18bb3f88b8aba986a3f774/packages/contract-artifacts/artifacts/IZeroEx.json)
* The Contract address (depends on the chain):[https://docs.0x.org/introduction/0x-cheat-sheet#exchange-proxy-addresses](https://docs.0x.org/introduction/0x-cheat-sheet#exchange-proxy-addresses)
