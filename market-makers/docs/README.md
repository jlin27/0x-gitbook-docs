---
description: >-
  The following docs and guides cover how liquidity providers such as Market
  Makers and AMMs can provide asset liquidity to 0x API
---

# Docs

### Docs

* [introduction.md](introduction.md "mention") - Learn how 0x API sources _off-chain_ liquidity from professional market makers via the _0x Request-for-Quote (“RFQ”) System_
* [providing-rfq-liquidity-to-0x-api.md](providing-rfq-liquidity-to-0x-api.md "mention") - If you represent a trading firm would like add liquidity to the 0x ecosystem via the RFQ system, apply here
* [providing-amm-liquidity-to-0x-api.md](providing-amm-liquidity-to-0x-api.md "mention") - To add your AMM as a liquidity source to the 0x API, apply here

### Guides

* [generating-0x-order-hashes.md](../guides/generating-0x-order-hashes.md "mention")
* [signing-0x-orders.md](../guides/signing-0x-orders.md "mention")
* [0x-v4-rfq-orders-code-examples-with-python.md](../guides/0x-v4-rfq-orders-code-examples-with-python.md "mention")​

The following are code examples from the [0x Starter Project](https://github.com/0xProject/0x-starter-project) which showcases a number of scenarios using the 0x v4 protocol. These are all runnable from the command-line:

* [Fill ERC20 RFQ order](https://github.com/0xProject/0x-starter-project/blob/master/src/scenarios/fill\_erc20\_rfq\_order.ts)
* [Fill ERC20 RFQ order with maker order signer](https://github.com/0xProject/0x-starter-project/blob/master/src/scenarios/fill\_erc20\_rfq\_order\_with\_maker\_order\_signer.ts)
* [Subscribe to RFQ order fill events](https://github.com/0xProject/0x-starter-project/blob/master/src/scenarios/fill\_erc20\_limit\_order.ts)
* [Execute Metatransaction](https://github.com/0xProject/0x-starter-project/blob/master/src/scenarios/execute\_metatransaction\_fill\_rfq\_order.ts) to fill RFQ order&#x20;
* [Fill ERC20 OTC order](https://github.com/0xProject/0x-starter-project/blob/master/src/scenarios/fill\_erc20\_otc\_order.ts)
* [(Advanced) Fill an aggregated quote via TransformERC20](https://github.com/0xProject/0x-starter-project/blob/master/src/scenarios/transform\_erc20.ts)
