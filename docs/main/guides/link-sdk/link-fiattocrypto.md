---
id: "link-fiat-to-crypto"
title: "Link.fiatToCrypto"
slug: "/link-fiat-to-crypto"
sidebar_position: 11
keywords: [imx-wallets]
---

The exchange functionality is available in SDK v1.3.33+ and is a collaboration between ImmutableX and Moonpay. `cryptoCurrencies` param is available in SDK v1.3.38+

The exchange process allows users to buy crypto from the ImmutableX platform using a credit card. There's no gas price for purchases, only a [Moonpay transaction fee](https://support.moonpay.com/hc/en-gb/articles/360011930117-What-fees-do-you-charge-).

The deposit happens directly on L2 via the transfer from Moonpay accounts.

The amount, fee and currency are selected within the Moonpay widget, and are under user's control.

:::info Minimal deposit amount
There's a minimal deposit amount of $20USD.
:::

The current available cryptocurrencies are:

- USDC
- ETH

The list of supported `cryptoCurrencies` in the Link SDK:

- `ExchangeCurrency.USDC`
- `ExchangeCurrency.ETH`

:::danger Exchange requires authenticated user
`Link.fiatToCrypto({})` should only be called when user is authenticated and logged in, otherwise it will require user to reconnect
:::

To initialize the exchange process with an option to choose any available currency, marketplace needs to call the exchange function:

```typescript
await link.fiatToCrypto({})
```

This displays the Link UI with loaded Moonpay widget:

![Exchange without parameters](/img/link-sdk-moonpay/exchange-without-params.png 'Exchange without parameters')

To initialize the exchange process for a specific currency:

```typescript
await link.fiatToCrypto({ cryptoCurrencies: ['ETH'] })
```

This displays the Link UI with loaded Moonpay widget:

![Exchange with specific currency](/img/link-sdk-moonpay/exchange-with-currency-chosen.png 'Exchange with specific currency')

To initialize the exchange process when a user can choose only specific currencies:

```typescript
await link.fiatToCrypto({ cryptoCurrencies: ['ETH', 'USDC'] })
// or
await link.fiatToCrypto({ cryptoCurrencies: ['IMX', 'GODS'] })
```

:::success Testing the flow in Sandbox
To test the transactions in the Sandbox test environment, please use the following test cards:
**CARD**: Visa
**NUMBER**: 4444493318246892
**DATE**: 12/2023
**CVC**: 123

**CARD**: Mastercard
**NUMBER**: 2222755234426838
**DATE**: 01/2024
**CVC**: 123
:::

## Errors

See error responses [here](./link-errors.md#fiat-to-crypto).