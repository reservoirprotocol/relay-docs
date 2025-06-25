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

To swap tokens across chains, you need to specify the source and destination chains, currencies, and the amount to swap. Use the `/execute/swap` endpoint to get comprehensive swap quotes that include optimal routing and fee calculations.

### Basic Cross-Chain Swap Quote

<CodeGroup>

```bash cURL
curl -X POST "https://api.relay.link/execute/swap" \
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
const response = await fetch('https://api.relay.link/execute/swap', {
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

response = requests.post('https://api.relay.link/execute/swap', json={
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

| Parameter              | Type    | Required | Description                                         |
| ---------------------- | ------- | -------- | --------------------------------------------------- |
| `user`                 | string  | Yes      | Address that will initiate the swap                 |
| `originChainId`        | number  | Yes      | Source chain ID (e.g., 1 for Ethereum)              |
| `destinationChainId`   | number  | Yes      | Destination chain ID (e.g., 8453 for Base)          |
| `originCurrency`       | string  | Yes      | Input currency ID (e.g., "eth", "usdc")             |
| `destinationCurrency`  | string  | Yes      | Output currency ID                                  |
| `amount`               | string  | Yes      | Amount in wei/smallest unit                         |
| `tradeType`            | string  | Yes      | "EXACT_INPUT", "EXACT_OUTPUT", or "EXPECTED_OUTPUT" |
| `recipient`            | string  | No       | Recipient address (defaults to user)                |
| `slippageTolerance`    | string  | No       | Slippage in basis points (e.g., "50" for 0.5%)      |
| `useExternalLiquidity` | boolean | No       | Use canonical\+ bridging for more liquidity         |

### Trade Types Explained

| Trade Type        | Description                                      | Use Case                                      |
| ----------------- | ------------------------------------------------ | --------------------------------------------- |
| `EXACT_INPUT`     | Specify exact input amount, get estimated output | "I want to swap exactly 0.1 ETH"              |
| `EXACT_OUTPUT`    | Specify exact output amount, get required input  | "I want to receive at least 100 USDC or more" |
| `EXPECTED_OUTPUT` | Specify expected output with slippage protection | "I expect ~100 USDC with slippage tolerance"  |

## Execute the Swap

After receiving a swap quote, execute it by processing each step in the response. The execution handles both the origin chain transaction and destination chain fulfillment.

### Process Swap Execution

<CodeGroup>

```javascript JavaScript/Node.js
async function executeCrossChainSwap(swapQuote, web3Provider) {
  console.log(`Swapping ${swapQuote.details.currencyIn.amountFormatted} ${swapQuote.details.currencyIn.currency.symbol} to ${swapQuote.details.currencyOut.amountFormatted} ${swapQuote.details.currencyOut.currency.symbol}`);
  
  for (const step of swapQuote.steps) {
    console.log(`Processing step: ${step.action}`);
    
    for (const item of step.items) {
      if (item.status === 'incomplete') {
        try {
          // Submit transaction
          const txHash = await submitTransaction(item.data, web3Provider);
          console.log(`Transaction submitted: ${txHash}`);
          
          // Wait for confirmation
          await waitForConfirmation(txHash, web3Provider);
          
          // Monitor swap progress
          if (item.check) {
            const result = await pollSwapCompletion(item.check);
            console.log(`Received ${result.details.currencyOut.amountFormatted} ${result.details.currencyOut.currency.symbol}`);
          }
          
        } catch (error) {
          console.error(`Swap failed:`, error);
          throw error;
        }
      }
    }
  }
}

async function pollSwapCompletion(checkInfo) {
  const maxAttempts = 60; // 1 minute with 1-second intervals
  
  for (let attempt = 0; attempt < maxAttempts; attempt++) {
    try {
      const response = await fetch(`https://api.relay.link${checkInfo.endpoint}`, {
        method: checkInfo.method
      });
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}: ${response.statusText}`);
      }
      
      const status = await response.json();
      
      if (status.status === 'success') {
        console.log('Cross-chain swap completed successfully');
        return status;
      } else if (status.status === 'failure') {
        throw new Error(`Swap failed: ${status.details || 'Unknown error'}`);
      } else if (status.status === 'refund') {
        console.warn('Swap was refunded due to market conditions');
        return status;
      }
      
      // Log progress
      if (attempt % 10 === 0) { // Every 10 seconds
        console.log(`Swap in progress... (${attempt}s elapsed)`);
      }
      
    } catch (error) {
      console.error(`Status check failed (attempt ${attempt + 1}):`, error);
    }
    
    await new Promise(resolve => setTimeout(resolve, 1000));
  }
  
  throw new Error('Swap timed out after 1 minute');
}

// Helper function to calculate price impact
function calculatePriceImpact(swapQuote) {
  const { totalImpact, swapImpact } = swapQuote.details;
  
  return {
    totalImpactPercent: parseFloat(totalImpact.percent),
    swapImpactPercent: parseFloat(swapImpact.percent),
    totalImpactUsd: parseFloat(totalImpact.usd),
    isHighImpact: Math.abs(parseFloat(totalImpact.percent)) > 3 // > 3% impact
  };
}
```


```python Python
import requests
import time

def execute_cross_chain_swap(swap_quote, web3_instance):
    currency_in = swap_quote['details']['currencyIn']
    currency_out = swap_quote['details']['currencyOut']
    
    print(f"Swapping {currency_in['amountFormatted']} {currency_in['currency']['symbol']} to {currency_out['amountFormatted']} {currency_out['currency']['symbol']}")
    
    for step in swap_quote['steps']:
        print(f"Processing step: {step['action']}")
        
        for item in step['items']:
            if item['status'] == 'incomplete':
                try:
                    # Submit transaction
                    tx_hash = submit_transaction(item['data'], web3_instance)
                    print(f"Transaction submitted: {tx_hash.hex()}")
                    
                    # Wait for confirmation
                    wait_for_confirmation(tx_hash, web3_instance)
                    
                    # Monitor swap progress
                    if 'check' in item:
                        result = poll_swap_completion(item['check'])
                        print(f"Received {result['details']['currencyOut']['amountFormatted']} {result['details']['currencyOut']['currency']['symbol']}")
                        
                except Exception as error:
                    print(f"Swap failed: {error}")
                    raise

def poll_swap_completion(check_info):
    max_attempts = 60  # 1 minute with 1-second intervals
    
    for attempt in range(max_attempts):
        try:
            response = requests.request(
                check_info['method'],
                f"https://api.relay.link{check_info['endpoint']}"
            )
            response.raise_for_status()
            status = response.json()
            
            if status['status'] == 'success':
                print('Cross-chain swap completed successfully')
                return status
            elif status['status'] == 'failure':
                raise Exception(f"Swap failed: {status.get('details', 'Unknown error')}")
            elif status['status'] == 'refund':
                print('Swap was refunded due to market conditions')
                return status
            
            # Log progress
            if attempt % 10 == 0:  # Every 10 seconds
                print(f"Swap in progress... ({attempt}s elapsed)")
                
        except Exception as error:
            print(f"Status check failed (attempt {attempt + 1}): {error}")
        
        time.sleep(1)
    
    raise Exception('Swap timed out after 1 minute')

def calculate_price_impact(swap_quote):
    total_impact = swap_quote['details']['totalImpact']
    swap_impact = swap_quote['details']['swapImpact']
    
    return {
        'total_impact_percent': float(total_impact['percent']),
        'swap_impact_percent': float(swap_impact['percent']),
        'total_impact_usd': float(total_impact['usd']),
        'is_high_impact': abs(float(total_impact['percent'])) > 3  # > 3% impact
    }
```

</CodeGroup>

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

## Advanced Swapping Features

### Price Impact Analysis

Before executing swaps, analyze price impact to protect users:

<CodeGroup>

```javascript JavaScript
function analyzeSwapQuote(swapQuote) {
  const impact = calculatePriceImpact(swapQuote);
  const rate = parseFloat(swapQuote.details.rate);
  
  console.log(`Exchange rate: 1 ${swapQuote.details.currencyIn.currency.symbol} = ${rate} ${swapQuote.details.currencyOut.currency.symbol}`);
  console.log(`Total impact: ${impact.totalImpactPercent.toFixed(2)}%`);
  console.log(`Swap impact: ${impact.swapImpactPercent.toFixed(2)}%`);
  
  if (impact.isHighImpact) {
    console.warn('⚠️ High price impact detected! Consider reducing swap size.');
    return { proceed: false, reason: 'High price impact' };
  }
  
  return { proceed: true };
}
```


```python Python
def analyze_swap_quote(swap_quote):
    impact = calculate_price_impact(swap_quote)
    rate = float(swap_quote['details']['rate'])
    
    currency_in = swap_quote['details']['currencyIn']['currency']['symbol']
    currency_out = swap_quote['details']['currencyOut']['currency']['symbol']
    
    print(f"Exchange rate: 1 {currency_in} = {rate} {currency_out}")
    print(f"Total impact: {impact['total_impact_percent']:.2f}%")
    print(f"Swap impact: {impact['swap_impact_percent']:.2f}%")
    
    if impact['is_high_impact']:
        print('⚠️ High price impact detected! Consider reducing swap size.')
        return {'proceed': False, 'reason': 'High price impact'}
    
    return {'proceed': True}
```

</CodeGroup>

### Custom Slippage Protection

Set custom slippage tolerance for volatile market conditions:

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

### App Fee Integration

Monetize your swap integration with app fees:

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

## Preflight Checklist

☐ **Currency support** - Verify both currencies are supported on their respective chains\\

☐ **Balance validation** - Ensure user has sufficient balance for swap amount \+ fees\\

☐ **Price impact check** - Analyze swap impact and warn users of high slippage\\

☐ **Slippage tolerance** - Set appropriate slippage for market conditions\\

☐ **Rate validation** - Compare rates with other sources to ensure competitiveness\\

☐ **Error handling** - Implement robust error handling for failed swaps\\

☐ **Progress monitoring** - Track swap status and provide user feedback\\

☐ **Market conditions** - Consider current market volatility when setting parameters\\

☐ **Gas optimization** - Monitor gas costs and suggest optimal timing

## API Base URLs

- **Mainnet**: `https://api.relay.link`
- **Testnet**: `https://api.testnets.relay.link`

Cross-chain swapping enables users to seamlessly exchange tokens across different blockchains while maintaining optimal pricing and minimal friction.