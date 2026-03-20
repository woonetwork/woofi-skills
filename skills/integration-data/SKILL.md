---
description: Query WOOFi integration data including trading pairs, ticker prices, and pool states for third-party integrations
triggers:
  - "trading pairs"
  - "ticker data"
  - "pool state"
  - "pool states"
  - "integration pairs"
  - "integration tickers"
  - "last price"
  - "woofi pools"
  - "oracle state"
  - "fee rate"
  - "pool reserves"
---

# WOOFi Integration Data Skill

## Overview

This skill queries the WOOFi integration API endpoints to retrieve trading pairs, 24-hour ticker data, and internal pool states. These are primarily used for third-party integrations (e.g., CoinMarketCap, CoinGecko) and for inspecting WOOFi liquidity pool internals.

**Base URL**: `https://api.woofi.com`
**Authentication**: None required
**Rate Limit**: 5 requests/second

---

## Endpoints

### 1. GET /integration/pairs

Returns supported trading pairs.

**Caching**: 3600 seconds (1 hour)

**Parameters**: None

**Response Fields**:

| Field | Type | Description |
|-------|------|-------------|
| `ticker_id` | string | Pair identifier (e.g., `ETH_USDC`) |
| `base` | string | Base token symbol |
| `target` | string | Quote token symbol |

---

### 2. GET /integration/tickers

Returns 24-hour market data for all integration pairs.

**Caching**: 5 seconds

**Parameters**: None

**Response Fields**:

| Field | Type | Description |
|-------|------|-------------|
| `ticker_id` | string | Pair identifier |
| `last_price` | number | Last traded price |
| `base_volume` | number | 24-hour volume in base token |
| `target_volume` | number | 24-hour volume in quote token |

**Logic**: Aggregates data from volume subgraphs across all supported chains.

---

### 3. GET /integration/pool_states

Returns the internal state of WOOFi Liquidity Pools (WooPP), including reserves, fees, prices, and oracle parameters.

**Caching**: 5 seconds

**Parameters**:

| Parameter | Required | Values |
|-----------|----------|--------|
| `network` | No | Network name (e.g., `arbitrum`, `bsc`) |

**Response Fields** (per token pool):

| Field | Type | Description |
|-------|------|-------------|
| `reserve` | string | Token reserve amount |
| `fee_rate` | number | Trading fee rate |
| `max_gamma` | number | Maximum gamma parameter |
| `max_notional_swap` | number | Maximum notional swap size |
| `cap_bal` | string | Balance cap |
| `price` | number | Current oracle price |
| `spread` | number | Price spread |
| `coeff` | number | Price coefficient |
| `wo_feasible` | boolean | Whether the oracle state is feasible |

**Implementation detail**: Uses `Multicall3` to batch-call `tokenInfos` on `WooPP` and `state` on `Wooracle` contracts.

---

## Example Queries

### List all trading pairs
```
GET https://api.woofi.com/integration/pairs
```

### Get 24h ticker data
```
GET https://api.woofi.com/integration/tickers
```

### Get pool states on Arbitrum
```
GET https://api.woofi.com/integration/pool_states?network=arbitrum
```

---

## Response Examples

### /integration/pairs
```json
{
  "status": "ok",
  "data": [
    {"ticker_id": "ETH_USDC", "base": "ETH", "target": "USDC"},
    {"ticker_id": "BTC_USDC", "base": "BTC", "target": "USDC"}
  ]
}
```

### /integration/pool_states
```json
{
  "status": "ok",
  "data": [
    {
      "symbol": "ETH",
      "reserve": "500000000000000000000",
      "fee_rate": 0.00025,
      "max_gamma": 0.01,
      "max_notional_swap": 1000000,
      "cap_bal": "1000000000000000000000",
      "price": 3500.25,
      "spread": 0.001,
      "coeff": 0.00001,
      "wo_feasible": true
    }
  ]
}
```

---

## Common Use Cases

1. **Price feed**: Use `/integration/tickers` for latest prices and 24h volume per pair
2. **Pair discovery**: Use `/integration/pairs` to list all tradeable pairs
3. **Pool health monitoring**: Use `/integration/pool_states` to check reserves, feasibility, and fee rates
4. **Oracle analysis**: Inspect `price`, `spread`, and `coeff` to understand WOOFi's pricing model
5. **Liquidity assessment**: Compare `reserve` against `cap_bal` to gauge pool utilization
