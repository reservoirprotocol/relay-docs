---
title: "Bridging"
description: "How to implement token bridging"
---

### Summary

<Steps>
  <Step title="Get a Quote">
    <a href="#get-a-quote" style={{ position:"absolute",top:10,left:10,right:10,bottom:10,borderBottom:0 }} />

    In this step we'll request a quote that includes all fees, transaction data, and execution steps required to bridge assets.
  </Step>
  <Step title="Execute the Bridge">
    <a href="#execute-the-bridge" style={{ position:"absolute",top:10,left:10,right:10,bottom:10,borderBottom:0 }} />

    Once we have the quote, we'll execute the bridge by submitting transactions and monitoring progress.
  </Step>
</Steps>

## Get a Quote

To bridge assets, you first need to get a quote that provides all necessary information including fees, transaction data, and execution steps. Use the [quote endpoint](/references/api/get-quote) to request bridging information.

### Basic Quote Request

<CodeGroup>

```bash cURL
curl -X POST "https://api.relay.link/quote" \
  -H "Content-Type: application/json" \
  -d '{
    "user": "0x742d35Cc6634C0532925a3b8D9d4DB0a2D7DD5B3",
    "originChainId": 1,
    "destinationChainId": 8453,
    "originCurrency": "0x0000000000000000000000000000000000000000",
    "destinationCurrency": "0x0000000000000000000000000000000000000000",
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
    originCurrency: '0x0000000000000000000000000000000000000000',
    destinationCurrency: '0x0000000000000000000000000000000000000000',
    amount: '100000000000000000', // 0.1 ETH in wei
    tradeType: 'EXACT_INPUT'
  })
});

const quote = await response.json();
```


```python Python
import requests

response = requests.post('https://api.relay.link/quote', json={
    'user': '0x742d35Cc6634C0532925a3b8D9d4DB0a2D7DD5B3',
    'originChainId': 1,
    'destinationChainId': 8453,
    'originCurrency': '0x0000000000000000000000000000000000000000',
    'destinationCurrency': '0x0000000000000000000000000000000000000000',
    'amount': '100000000000000000',  # 0.1 ETH in wei
    'tradeType': 'EXACT_INPUT'
})

quote = response.json()
```

</CodeGroup>

### Quote Response Structure

The quote response contains all information needed to execute the bridge:

```json
{
  "steps": [
    {
      "id": "deposit",
      "action": "Confirm transaction in your wallet",
      "description": "Deposit funds for executing the bridge",
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
      "amount": "21000000000000000",
      "currency": "eth"
    },
    "relayer": {
      "amount": "5000000000000000",
      "currency": "eth"
    }
  },
  "details": {
    "operation": "bridge",
    "timeEstimate": 30,
    "currencyIn": {
      "currency": {
        "chainId": 1,
        "address": "0x0000000000000000000000000000000000000000",
        "symbol": "ETH",
        "name": "Ethereum",
        "decimals": 18
      },
      "amount": "100000000000000000"
    },
    "currencyOut": {
      "currency": {
        "chainId": 8453,
        "address": "0x0000000000000000000000000000000000000000",
        "symbol": "ETH",
        "name": "Ethereum",
        "decimals": 18
      },
      "amount": "95000000000000000"
    }
  }
}
```

### Quote Parameters

| Parameter              | Type    | Required | Description                                                                                                                                                                       |
| ---------------------- | ------- | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `user`                 | string  | Yes      | Address that will deposit funds and submit transactions                                                                                                                           |
| `originChainId`        | number  | Yes      | Source chain ID (e.g., 1 for Ethereum)                                                                                                                                            |
| `destinationChainId`   | number  | Yes      | Destination chain ID (e.g., 8453 for Base)                                                                                                                                        |
| `originCurrency`       | string  | Yes      | Currency contract on source chain (e.g., "0x0000000000000000000000000000000000000000", "0x833589fcd6edb6e08f4c7c32d4f71b54bda02913")                                              |
| `destinationCurrency`  | string  | Yes      | Currency contract on destination chain                                                                                                                                            |
| `amount`               | string  | Yes      | Amount in wei/smallest unit                                                                                                                                                       |
| `tradeType`            | string  | Yes      | "EXACT_INPUT" or "EXACT_OUTPUT"                                                                                                                                                   |
| `recipient`            | string  | No       | Recipient address (defaults to user)                                                                                                                                              |
| `slippageTolerance`    | string  | No       | Slippage in basis points (e.g., "50" for 0.5%)                                                                                                                                    |
| `useExternalLiquidity` | boolean | No       | Use canonical\+ bridging for more liquidity                                                                                                                                       |
| `referrer`             | string  | No       | Identifier that can be used to monitor transactions from a specific source.                                                                                                       |
| `refundTo`             | string  | No       | Address to send the refund to in the case of failure, if not specified the user address is used                                                                                   |
| `topupGas`             | boolean | No       | If set, the destination fill will include a gas topup to the recipient (only supported for EVM chains if the requested currency is not the gas currency on the destination chain) |
| `topupGasAmount`       | string  | No       | The destination gas topup amount in USD decimal format, e.g 100000 = \$1. topupGas is required to be enabled. Defaults to 2000000 (\$2)                                           |

## Execute the Bridge

After receiving a bridge quote, execute it by processing each step in the response. The execution handles both the origin chain transaction and destination chain fulfillment.\
\
Learn more about step execution using the API [here](/references/api/step-execution).

### Monitor Bridge Status

You can check the status of a bridge operation at any time using the `/intents/status` endpoint:

<CodeGroup>

```bash cURL
curl "https://api.relay.link/intents/status?requestId=0x92b99e6e1ee1deeb9531b5ad7f87091b3d71254b3176de9e8b5f6c6d0bd3a331"
```


```javascript JavaScript
const checkStatus = async (requestId) => {
  const response = await fetch(
    `https://api.relay.link/intents/status?requestId=${requestId}`
  );
  return await response.json();
};

// Example response
{
  "status": "success",
  "inTxHashes": ["0xe53021eaa..."],
  "txHashes": ["0x9da7bc54df..."],
  "time": 1713290386145,
  "originChainId": 1,
  "destinationChainId": 7777777
}
```

</CodeGroup>

### Status Values

| Status    | Description                                     |
| --------- | ----------------------------------------------- |
| `waiting` | Deposit tx for the request is yet to be indexed |
| `pending` | Deposit tx was indexed, now the fill is pending |
| `success` | Relay completed successfully                    |
| `failure` | Relay failed                                    |
| `refund`  | Funds were refunded due to failure              |

## Advanced Features

### App Fees

You can include app fees in your bridge requests to monetize your integration:

```json
{
  "user": "0x742d35Cc6634C0532925a3b8D9d4DB0a2D7DD5B3",
  "originChainId": 1,
  "destinationChainId": 8453,
  "originCurrency": "eth",
  "destinationCurrency": "eth",
  "amount": "100000000000000000",
  "tradeType": "EXACT_INPUT",
  "appFees": [
    {
      "recipient": "0x742d35Cc6634C0532925a3b8D9d4DB0a2D7DD5B3",
      "fee": "100"  // 1% in basis points
    }
  ]
}
```

### Custom Slippage

Control slippage tolerance for your bridges:

```json
{
  "user": "0x742d35Cc6634C0532925a3b8D9d4DB0a2D7DD5B3",
  "originChainId": 1,
  "destinationChainId": 8453,
  "originCurrency": "eth",
  "destinationCurrency": "eth",
  "amount": "100000000000000000",
  "tradeType": "EXACT_INPUT",
  "slippageTolerance": "50"  // 0.5% slippage tolerance
}
```

## Preflight Checklist

☐ **Verify user balance** - Ensure the user has sufficient funds for the bridge amount plus fees

☐ **Check chain support** - Confirm both origin and destination chains are supported

☐ **Validate quote** - Quotes expire after 30 seconds, so fetch fresh quotes before execution

☐ **Handle errors** - Implement proper error handling for API requests and transaction failures

☐ **Monitor progress** - Use the status endpoints to track bridge completion