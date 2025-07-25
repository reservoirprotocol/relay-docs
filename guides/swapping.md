---
title: "Swapping"
description: "How to implement cross-chain token swaps"
---

### Summary

<Steps>
  <Step title="Get a Quote">
    <a href="#get-a-quote" style={{ position:"absolute",top:10,left:10,right:10,bottom:10,borderBottom:0 }} />

    In this step we'll request a quote that includes all fees, exchange rates, and execution steps required to swap tokens across chains.
  </Step>
  <Step title="Execute the Swap">
    <a href="#execute-the-swap" style={{ position:"absolute",top:10,left:10,right:10,bottom:10,borderBottom:0 }} />

    Once we have the quote, we'll execute the cross-chain swap by submitting transactions and monitoring progress.
  </Step>
</Steps>

## Get a Quote

To swap tokens across chains, you need to specify the source and destination chains, currencies, and the amount to swap. Use the [quote endpoint](/references/api/get-quote) to get comprehensive swap quotes that include optimal routing and fee calculations.

### Basic Cross-Chain Swap Quote

<CodeGroup>

```bash cURL
curl -X POST "https://api.relay.link/quote" \
  -H "Content-Type: application/json" \
  -d '{
    "user": "0x742d35Cc6634C0532925a3b8D9d4DB0a2D7DD5B3",
    "originChainId": 1,
    "destinationChainId": 8453,
    "originCurrency": "eth",
    "destinationCurrency": "usdc",
    "amount": "100000000000000000",
    "tradeType": "EXACT_INPUT"
  }'
```


```javascript JavaScript/Node.js
const response = await fetch('https://api.relay.link/quote', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    user: '0x742d35Cc6634C0532925a3b8D9d4DB0a2D7DD5B3',
    originChainId: 1,
    destinationChainId: 8453,
    originCurrency: 'eth',
    destinationCurrency: 'usdc',
    amount: '100000000000000000', // 0.1 ETH in wei
    tradeType: 'EXACT_INPUT'
  })
});

const swapQuote = await response.json();
```


```python Python
import requests

response = requests.post('https://api.relay.link/quote', json={
    'user': '0x742d35Cc6634C0532925a3b8D9d4DB0a2D7DD5B3',
    'originChainId': 1,
    'destinationChainId': 8453,
    'originCurrency': 'eth',
    'destinationCurrency': 'usdc',
    'amount': '100000000000000000',  # 0.1 ETH in wei
    'tradeType': 'EXACT_INPUT'
})

swap_quote = response.json()
```

</CodeGroup>

### Swap Quote Response Structure

The swap response contains detailed information about the exchange rates, fees, and execution steps:

```json
{
  "steps": [
    {
      "id": "deposit",
      "action": "Confirm transaction in your wallet",
      "description": "Depositing funds to the relayer to execute the swap for USDC",
      "kind": "transaction",
      "requestId": "0x92b99e6e1ee1deeb9531b5ad7f87091b3d71254b3176de9e8b5f6c6d0bd3a331",
      "items": [
        {
          "status": "incomplete",
          "data": {
            "from": "0x742d35Cc6634C0532925a3b8D9d4DB0a2D7DD5B3",
            "to": "0xf70da97812cb96acdf810712aa562db8dfa3dbef",
            "data": "0x00fad611",
            "value": "100000000000000000",
            "chainId": 1
          },
          "check": {
            "endpoint": "/intents/status?requestId=0x92b99e6e1ee1deeb9531b5ad7f87091b3d71254b3176de9e8b5f6c6d0bd3a331",
            "method": "GET"
          }
        }
      ]
    }
  ],
  "fees": {
    "gas": {
      "currency": {
        "chainId": 1,
        "address": "0x0000000000000000000000000000000000000000",
        "symbol": "ETH",
        "name": "Ethereum",
        "decimals": 18
      },
      "amount": "21000000000000000",
      "amountFormatted": "0.021",
      "amountUsd": "84.21"
    },
    "relayer": {
      "currency": {
        "chainId": 8453,
        "address": "0x833589fcd6edb6e08f4c7c32d4f71b54bda02913",
        "symbol": "USDC",
        "name": "USD Coin",
        "decimals": 6
      },
      "amount": "5000000",
      "amountFormatted": "5.0",
      "amountUsd": "5.0"
    }
  },
  "details": {
    "operation": "swap",
    "timeEstimate": 30,
    "currencyIn": {
      "currency": {
        "chainId": 1,
        "address": "0x0000000000000000000000000000000000000000",
        "symbol": "ETH",
        "name": "Ethereum",
        "decimals": 18
      },
      "amount": "100000000000000000",
      "amountFormatted": "0.1",
      "amountUsd": "400.0"
    },
    "currencyOut": {
      "currency": {
        "chainId": 8453,
        "address": "0x833589fcd6edb6e08f4c7c32d4f71b54bda02913",
        "symbol": "USDC",
        "name": "USD Coin",
        "decimals": 6
      },
      "amount": "395000000",
      "amountFormatted": "395.0",
      "amountUsd": "395.0"
    },
    "rate": "3950.0",
    "totalImpact": {
      "usd": "-5.0",
      "percent": "-1.25"
    },
    "swapImpact": {
      "usd": "-2.5",
      "percent": "-0.625"
    }
  }
}
```

### Quote Parameters for Swapping

| Parameter              | Type    | Required | Description                                                                                                                                                                       |
| ---------------------- | ------- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `user`                 | string  | Yes      | Address that will initiate the swap                                                                                                                                               |
| `originChainId`        | number  | Yes      | Source chain ID (e.g., 1 for Ethereum)                                                                                                                                            |
| `destinationChainId`   | number  | Yes      | Destination chain ID (e.g., 8453 for Base)                                                                                                                                        |
| `originCurrency`       | string  | Yes      | Currency contract on source chain (e.g., "0x0000000000000000000000000000000000000000", "0x833589fcd6edb6e08f4c7c32d4f71b54bda02913")                                              |
| `destinationCurrency`  | string  | Yes      | Currency contract on destination chain                                                                                                                                            |
| `amount`               | string  | Yes      | Amount in wei/smallest unit                                                                                                                                                       |
| `tradeType`            | string  | Yes      | "EXACT_INPUT", "EXACT_OUTPUT", or "EXPECTED_OUTPUT"                                                                                                                               |
| `recipient`            | string  | No       | Recipient address (defaults to user)                                                                                                                                              |
| `slippageTolerance`    | string  | No       | Slippage in basis points (e.g., "50" for 0.5%)                                                                                                                                    |
| `useExternalLiquidity` | boolean | No       | Use canonical\+ bridging for more liquidity                                                                                                                                       |
| `referrer`             | string  | No       | Identifier that can be used to monitor transactions from a specific source.                                                                                                       |
| `refundTo`             | string  | No       | Address to send the refund to in the case of failure, if not specified the user address is used                                                                                   |
| `topupGas`             | boolean | No       | If set, the destination fill will include a gas topup to the recipient (only supported for EVM chains if the requested currency is not the gas currency on the destination chain) |
| `topupGasAmount`       | string  | No       | The destination gas topup amount in USD decimal format, e.g 100000 = \$1. topupGas is required to be enabled. Defaults to 2000000 (\$2)                                           |

### Trade Types Explained

| Trade Type        | Description                                      | Use Case                                      |
| ----------------- | ------------------------------------------------ | --------------------------------------------- |
| `EXACT_INPUT`     | Specify exact input amount, get estimated output | "I want to swap exactly 0.1 ETH"              |
| `EXACT_OUTPUT`    | Specify exact output amount, get required input  | "I want to receive at least 100 USDC or more" |
| `EXPECTED_OUTPUT` | Specify expected output with slippage protection | "I expect ~100 USDC with slippage tolerance"  |

## Execute the Swap

After receiving a swap quote, execute it by processing each step in the response. The execution handles both the origin chain transaction and destination chain fulfillment.\
\
Learn more about step execution using the API [here](/references/api/step-execution).

### Monitor Swap Status

Track your cross-chain swap progress using the status endpoint:

<CodeGroup>

```bash cURL
curl "https://api.relay.link/intents/status/v2?requestId=0x92b99e6e1ee1deeb9531b5ad7f87091b3d71254b3176de9e8b5f6c6d0bd3a331"
```


```javascript JavaScript
const checkSwapStatus = async (requestId) => {
  const response = await fetch(
    `https://api.relay.link/intents/status/v2?requestId=${requestId}`
  );
  
  if (!response.ok) {
    throw new Error(`Status check failed: ${response.statusText}`);
  }
  
  return await response.json();
};

// Example response for successful swap
{
  "status": "success",
  "inTxHashes": ["0xe53021eaa..."], // Origin chain deposit
  "txHashes": ["0x9da7bc54df..."], // Destination chain fulfillment
  "time": 1713290386145,
  "originChainId": 1,
  "destinationChainId": 8453
}
```

</CodeGroup>

### Status Values

| Status    | Description                                     |
| :-------- | :---------------------------------------------- |
| `waiting` | Deposit tx for the request is yet to be indexed |
| `pending` | Deposit tx was indexed, now the fill is pending |
| `success` | Relay completed successfully                    |
| `failure` | Relay failed                                    |
| `refund`  | Funds were refunded due to failure              |

## Advanced  Features

### App Fees

You can include app fees in your swap requests to monetize your integration:

```json
{
  "user": "0x742d35Cc6634C0532925a3b8D9d4DB0a2D7DD5B3",
  "originChainId": 1,
  "destinationChainId": 8453,
  "originCurrency": "eth",
  "destinationCurrency": "usdc",
  "amount": "100000000000000000",
  "tradeType": "EXACT_INPUT",
  "appFees": [
    {
      "recipient": "0xYourFeeRecipient",
      "fee": "25"
    }
  ]
}
```

### Custom Slippage

Control slippage tolerance for your swaps:

```json
{
  "user": "0x742d35Cc6634C0532925a3b8D9d4DB0a2D7DD5B3",
  "originChainId": 1,
  "destinationChainId": 8453,
  "originCurrency": "eth",
  "destinationCurrency": "usdc",
  "amount": "100000000000000000",
  "tradeType": "EXACT_INPUT",
  "slippageTolerance": "100"
}
```

## Preflight Checklist

☐ **Currency support** - Verify both currencies are supported on their respective chains

☐ **Balance validation** - Ensure user has sufficient balance for swap amount \+ fees

☐ **Price impact check** - Analyze swap impact and warn users of high slippage

☐ **Slippage tolerance** - Set appropriate slippage for market conditions

☐ **Progress monitoring** - Track swap status and provide user feedback