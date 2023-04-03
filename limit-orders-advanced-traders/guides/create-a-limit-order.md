# Create a Limit Order

When created with the 0x API, these limit orders are represented JSON objects (refer to [this documentation](https://protocol.0x.org/en/latest/basics/orders.html#limit-orders) for the order format). In order to create a valid 0x Limit Order with the 0x API, you can use the [@0x/protocol-utils ](https://github.com/0xProject/protocol/tree/development/packages/protocol-utils)TypeScript/Javascript library. The TypeScript library will help you:

1. Generate an order in the proper format
2. Generating a proper hash for the order contents
3. Sign the order hash using your keys (which makes the order valid)

Some basic code demonstrating how to create, sign and submit 0x Limit Orders on the Ropsten test network is available [here](https://codesandbox.io/s/recursing-bell-ydbxb).

### Code Examples

* **Typescript** - [ 0x Starter Project](https://github.com/0xProject/0x-starter-project) comes with a number of limit order scenarios including the [Create and sign a ERC20 limit order](https://github.com/0xProject/0x-starter-project/blob/master/src/scenarios/cancel\_pair\_limit\_orders.ts) scenario.\

* **Python** - [Python 0x v4 RFQ Guide](https://gist.github.com/PirosB3/8141b51fbb307bca265866ef1cef564f) covers how to create, hash, sign, fill, get state, and cancel a 0x RFQ Order. Note that the code can be modified by making minor modifications, to also work for [0x Limit Orders](https://protocol.0x.org/en/latest/basics/orders.html#limit-orders).&#x20;
