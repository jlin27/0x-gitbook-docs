# Table of contents

## 👋 Introduction

* [Welcome](README.md)
* [Introduction to 0x](introduction/introduction-to-0x.md)
* [Guides](introduction/guides.md)
* [0x Concept Videos](introduction/0x-concept-videos.md)
* [Community](introduction/community.md)
* [0x Cheat Sheet](introduction/0x-cheat-sheet.md)
* [FAQs & Troubleshooting](https://docs.0x.org/developer-resources/faqs-and-troubleshooting)

## 🔁 0x Swap API

* [Introduction](0x-swap-api/introduction.md)
* [API References](0x-swap-api/api-references/README.md)
  * [GET /swap/v1/quote](0x-swap-api/api-references/get-swap-v1-quote.md)
  * [GET /swap/v1/price](0x-swap-api/api-references/get-swap-v1-price.md)
  * [GET /swap/v1/sources](0x-swap-api/api-references/get-swap-v1-sources.md)
* [Advanced Topics](0x-swap-api/advanced-topics/README.md)
  * [Rate Limiting](0x-swap-api/advanced-topics/rate-limiting.md)
  * [How to Set Your Token Allowances](0x-swap-api/advanced-topics/how-to-set-your-token-allowances.md)
  * [Slippage Protection](0x-swap-api/advanced-topics/slippage-protection.md)
  * [Price Impact Protection](0x-swap-api/advanced-topics/price-impact-protection.md)
* [Guides](0x-swap-api/guides/README.md)
  * [Swap Tokens with 0x Swap API](0x-swap-api/guides/swap-tokens-with-0x-swap-api.md)
  * [How to Build a Token Swap Dapp With 0x API](https://docs.alchemy.com/alchemy/road-to-web3/weekly-learning-challenges/9.-how-to-build-a-token-swap-dapp-with-0x-api)
  * [Use 0x API Liquidity in Your Smart Contracts](0x-swap-api/guides/use-0x-api-liquidity-in-your-smart-contracts.md)
  * [Accessing RFQ liquidity on 0x API](0x-swap-api/guides/accessing-rfq-liquidity-on-0x-api.md)
  * [Troubleshooting 0x API Swaps](0x-swap-api/guides/troubleshooting-0x-api-swaps.md)
  * [Working in the Testnet](https://docs.0x.org/limit-orders-advanced-traders/guides/working-in-the-testnet)
* [FAQs & Troubleshooting](https://docs.0x.org/developer-resources/api-faqs)

## ⛽ Tx Relay API

* [Introduction](tx-relay-api/introduction.md)
* [Development Status](tx-relay-api/development-status.md)
* [API References](tx-relay-api/api-references.md)
* [Tx Relay FAQ](tx-relay-api/tx-relay-faq/README.md)
  * [Gasless Approvals Token List](tx-relay-api/tx-relay-faq/gasless-approvals-token-list.md)

## 🤖 0x Orderbook API

* [Introduction](0x-orderbook-api/introduction.md)
* [API References](0x-orderbook-api/api-references/README.md)
  * [GET /orderbook/v1/](0x-orderbook-api/api-references/get-orderbook-v1.md)
  * [GET /orderbook/v1/orders](0x-orderbook-api/api-references/get-orderbook-v1-orders.md)
  * [POST /orderbook/v1/order](0x-orderbook-api/api-references/post-orderbook-v1-order.md)
  * [POST /orderbook/v1/orders](0x-orderbook-api/api-references/post-orderbook-v1-orders.md)
  * [GET /orderbook/v1/order/:orderHash:](0x-orderbook-api/api-references/get-orderbook-v1-order-orderhash.md)
  * [POST /orderbook/v1/order\_config](0x-orderbook-api/api-references/post-orderbook-v1-order\_config.md)
  * [GET /orderbook/v1/fee\_recipients](0x-orderbook-api/api-references/get-orderbook-v1-fee\_recipients.md)
  * [Websocket API](0x-orderbook-api/api-references/websocket-api.md)
* [Rate Limiting](0x-orderbook-api/rate-limiting.md)
* [Connection Limit](0x-orderbook-api/connection-limit.md)

## 🤑 Limit Orders (Advanced Traders)

* [Docs](limit-orders-advanced-traders/docs/README.md)
  * [Introduction](limit-orders-advanced-traders/docs/introduction.md)
  * [Limit Order Structure](limit-orders-advanced-traders/docs/limit-order-structure.md)
  * [Monitoring 0x Limit Orders](limit-orders-advanced-traders/docs/monitoring-0x-limit-orders.md)
* [Guides](limit-orders-advanced-traders/guides/README.md)
  * [Create a Limit Order](limit-orders-advanced-traders/guides/create-a-limit-order.md)
  * [Cancel a Limit Order](limit-orders-advanced-traders/guides/cancel-a-limit-order.md)
  * [Fill a Limit Order](limit-orders-advanced-traders/guides/fill-a-limit-order.md)
  * [Limit Order Status](limit-orders-advanced-traders/guides/limit-order-status.md)
  * [Working in the Testnet](limit-orders-advanced-traders/guides/working-in-the-testnet.md)

## 💰 Market Makers

* [Docs](market-makers/docs/README.md)
  * [Introduction](market-makers/docs/introduction.md)
  * [RFQ Order Structure](market-makers/docs/rfq-order-structure.md)
  * [Providing RFQ liquidity to 0x API](market-makers/docs/providing-rfq-liquidity-to-0x-api.md)
  * [Providing AMM liquidity to 0x API](market-makers/docs/providing-amm-liquidity-to-0x-api.md)
* [Guides](market-makers/guides/README.md)
  * [Generating 0x Order Hashes](market-makers/guides/generating-0x-order-hashes.md)
  * [Signing 0x Orders](market-makers/guides/signing-0x-orders.md)
  * [0x v4 RFQ Orders Code Examples with Python](market-makers/guides/0x-v4-rfq-orders-code-examples-with-python.md)

## ✨ NFT Support

* [Docs](nft-support/docs/README.md)
  * [Introduction](nft-support/docs/introduction.md)
* [Guides](nft-support/guides/README.md)
  * [Creating Orders](nft-support/guides/creating-orders/README.md)
    * [Royalties and Fees](nft-support/guides/creating-orders/royalties-and-fees.md)
    * [Collection Offers](nft-support/guides/creating-orders/collection-offers.md)
  * [Signing Orders](nft-support/guides/signing-orders/README.md)
    * [On-chain Orders](nft-support/guides/signing-orders/on-chain-orders.md)
  * [Filling Orders](nft-support/guides/filling-orders.md)
  * [Cancelling Orders](nft-support/guides/cancelling-orders.md)
  * [Fetching NFT Order Data](nft-support/guides/fetching-nft-order-data.md)

## 🌐 Protocol

* [Docs](protocol/docs/README.md)
  * [Exchange Proxy](protocol/docs/exchange-proxy/README.md)
    * [Features](protocol/docs/exchange-proxy/features/README.md)
      * [NativeOrders](protocol/docs/exchange-proxy/features/nativeorders.md)
      * [ERC721Orders](protocol/docs/exchange-proxy/features/erc721orders.md)
      * [ERC1155Orders](protocol/docs/exchange-proxy/features/erc1155orders.md)
  * [Signatures](protocol/docs/signatures.md)

## ⚖ Governance

* [Docs](governance/docs/README.md)
  * [Introduction](governance/docs/introduction.md)
  * [Governance Participants](governance/docs/governance-participants.md)
  * [Types of Votes](governance/docs/types-of-votes.md)
  * [Community Treasury](governance/docs/community-treasury.md)
  * [Apply for a 0xDAO Grant (v2)](governance/docs/apply-for-a-0xdao-grant-v2.md)
* [Guides](governance/guides/README.md)
  * [How to Delegate ZRX](governance/guides/how-to-delegate-zrx.md)
  * [How to Vote](governance/guides/how-to-vote.md)

## 📚 Developer Resources

* [🚦 0x System Status](https://status.0x.org/)
* [📘 Glossary](developer-resources/glossary.md)
* [🤔 FAQs & Troubleshooting](developer-resources/faqs-and-troubleshooting.md)
* [📃 Contract Addresses](developer-resources/contract-addresses.md)
* [🐙 Repositories](developer-resources/repositories.md)
* [📊 How to Get 0x and Matcha Data](developer-resources/how-to-get-0x-and-matcha-data.md)
* [🔎 Audits](developer-resources/audits.md)
* [🐛 Bounties](developer-resources/bounties.md)
* [🧑⚖ 0x Legal Guide](developer-resources/0x-legal-guide.md)
* [📄 White Paper](developer-resources/white-paper.md)
