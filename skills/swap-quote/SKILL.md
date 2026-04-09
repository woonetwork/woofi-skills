---
description: Get token exchange rates and expected receive amounts without executing a trade.
triggers:
  - "quote"
  - "price"
  - "exchange rate"
  - "token price"
  - "how much will I get"
---

# WOOFi Swap Quote Skill

## Overview

This skill queries the WOOFi `v2/quote` API to get aggregated token exchange rates and the expected received token amount for a specific chain. It does not generate or execute any transaction data.

**Base URL**: `https://sapi.woofi.com`
**Endpoint**: `POST /v2/quote`
**Content-Type**: `application/json`

---

## Parameters (QuoteRequest)

All parameters are passed in the JSON body.

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `chain` | `int \| str` | Yes | - | Chain ID (`42161`) or chain name (`"arbitrum"`). See chain name resolution below. |
| `sell_token`| `str` | Yes | - | Token address (`"0xaf88..."`) or token symbol/name (`"USDC"`). Native ETH/BNB/etc resolves from symbol. |
| `buy_token` | `str` | Yes | - | Token address or symbol/name of token to buy |
| `sell_amount`| `str` | Yes | - | Human-readable amount to sell (e.g., `"1.5"`) |
| `slippage_pct`| `float` | No | `0.5` | Max allowed slippage percentage |
| `woofi_only`| `bool` | No | `false` | If `true`, only sources quotes directly from WOOFi pools |

### Chain Name Resolution

Case-insensitive. Automatically strips spaces, dashes, underscores, and "chain"/"network" suffixes.

| Chain ID | Accepted Names |
|----------|---------------|
| 1 | ethereum, eth, mainnet |
| 10 | optimism, op |
| 56 | bsc, bnb, binance, bnbchain, binancesmartchain |
| 137 | polygon, matic, pol |
| 143 | monad |
| 146 | sonic |
| 324 | zksync, zksyncera |
| 999 | hyperevm, hyper |
| 5000 | mantle |
| 8453 | base |
| 42161 | arbitrum, arb, arbi, arbitrumone |
| 43114 | avalanche, avax |
| 59144 | linea |
| 80094 | berachain, bera |

### Token Name Resolution

Tokens can be specified by address or symbol/name. Resolution layers:
1. **Local symbol index** — common tokens: WETH, USDC, USDT, WBTC, etc.
2. **Static JSON manifests** — 30+ tokens per chain, indexed by symbol and name
3. **Alias table** — native tokens and wrapped variants (e.g., `eth` -> WETH, `btc` -> WBTC)
4. **External API fallback** (async) — queries Odos + 1inch APIs in parallel, results cached

**Native tokens**: `"eth"` on Arbitrum, `"matic"` on Polygon, etc. resolve to `0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE`.

---

## Response Fields (QuoteResponse)

| Field | Type | Description |
|-------|------|-------------|
| `chain` | `int` | Chain ID |
| `sell_token` | `str` | Resolved sell token address |
| `buy_token` | `str` | Resolved buy token address |
| `sell_amount` | `str` | Sell amount requested |
| `buy_amount` | `str` | Expected buy amount |
| `price` | `str` | Current execution price |
| `guaranteed_price` | `str` | Minimum guaranteed price after considering slippage |

---

## Example Usage

### Getting a Quote for swapping USDC to WBTC on Arbitrum
```bash
curl -X POST "https://sapi.woofi.com/v2/quote" \
  -H "Content-Type: application/json" \
  -d '{
    "chain": "arbitrum",
    "sell_token": "USDC",
    "buy_token": "WBTC",
    "sell_amount": "1000"
  }'
```

### Response Example
```json
{
  "chain": 42161,
  "sell_token": "0xaf88d065e77c8cC2239327C5EDb3A432268e5831",
  "buy_token": "0x2f2a2543b76a4166549f7aab2e75bef0aefc5b0f",
  "sell_amount": "1000",
  "buy_amount": "0.01524300",
  "price": "0.000015243",
  "guaranteed_price": "0.000015167"
}
```

### Using chain ID instead of name
```bash
curl -X POST "https://sapi.woofi.com/v2/quote" \
  -H "Content-Type: application/json" \
  -d '{
    "chain": 42161,
    "sell_token": "0xaf88d065e77c8cC2239327C5EDb3A432268e5831",
    "buy_token": "0x2f2a2543b76a4166549f7aab2e75bef0aefc5b0f",
    "sell_amount": "1000"
  }'
```

---

## Common Use Cases

1. **Price Checking**: Determine the current value of a token swap.
2. **Slippage Evaluation**: Check the `guaranteed_price` against the standard `price` based on the provided `slippage_pct`.
3. **Pre-trade verification**: Always perform a quote before initiating a `swap` execution to verify expectations.
