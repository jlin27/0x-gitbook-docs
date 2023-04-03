# GET /orderbook/v1/fee\_recipients

Retrieves a list of valid feeRecipient addresses.

{% hint style="warning" %}
This endpoint is paginated. By default, a request returns page 1 with 20 orders. It is possible to change which page and how many records are returned per page via the query parameters. See[#pagination](./#pagination "mention")for more details.&#x20;
{% endhint %}

## Request

No request params.

## Response

### Record

A valid Ethereum address(s).

## Examples

### Get Fee Recipients

#### **Request**

GET `/orderbook/v1/fee_recipients`

#### **Request**

```
{
    "total": 1,
    "page": 1,
    "perPage": 20,
    "records": ["0x0000000000000000000000000000000000000000"]
}
```
