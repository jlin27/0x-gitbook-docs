# GET /swap/v1/sources

Returns the liquidity sources enabled for the chain.

## Request

No request parameters.

## Response

An array of liquidity sources.

## Example

### Get a list of liquidity sources on Fantom

**Request**

```
https://fantom.api.0x.org/swap/v1/sources
```

**Response**

```
{
    "records": [
        "Beethovenx",
        "Curve",
        "Curve_V2",
        "JetSwap",
        "MorpheusSwap",
        "MultiHop",
        "SpiritSwap",
        "SpookySwap",
        "SushiSwap",
        "Synapse"
    ]
}
```
