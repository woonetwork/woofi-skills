---
description: Query WOOFi Swap trading statistics including volume, trader counts, and transaction counts by time period and network
triggers:
  - "trading stats"
  - "trading volume"
  - "woofi stats"
  - "swap volume"
  - "how much volume"
  - "trader count"
  - "transaction count"
  - "woofi trading data"
---

# WOOFi Trading Statistics Skill

## Overview

This skill queries the WOOFi Swap API `/stat` endpoint to retrieve trading statistics across supported blockchain networks and time periods.

**Base URL**: `https://api.woofi.com`
**Endpoint**: `GET /stat`
**Authentication**: None required
**Rate Limit**: 120 requests/minute

---

## Parameters

| Parameter | Required | Values |
|-----------|----------|--------|
| `period`  | Yes | `1d`, `1w`, `1m`, `3m`, `1y`, `all` |
| `network` | Yes | See supported networks below |

### Supported Networks

`bsc`, `avax`, `polygon`, `arbitrum`, `optimism`, `linea`, `base`, `mantle`, `sonic`, `berachain`, `hyperevm`, `monad`, `solana`

### Period Behavior

- `1d` returns **hourly** buckets
- `1w`, `1m`, `3m`, `1y`, `all` return **daily** buckets
- Prefer `period=1m` with UTC timestamp filtering over `period=1w` for complete weekly data

---

## Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `timestamp` | string | UTC timestamp for the bucket |
| `volume_usd` | string | Volume in wei — divide by `10^18` to get USD value |
| `trader_count` | integer | Unique traders in this bucket |
| `tx_count` | integer | Number of transactions in this bucket |

---

## Critical Implementation Notes

**Do NOT sum `trader_count` across buckets.** Each bucket represents unique traders within that time window; summing them inflates the unique user count. To get unique traders for a period, use the last bucket or a dedicated aggregate query.

**Volume conversion**: All `volume_usd` values are in wei format. Convert with:
```
actual_usd_volume = volume_usd / 10^18
```

---

## Example Queries

### Get last 24h volume on Arbitrum
```
GET https://api.woofi.com/stat?period=1d&network=arbitrum
```

### Get monthly stats on BSC
```
GET https://api.woofi.com/stat?period=1m&network=bsc
```

### Cross-chain analysis
For cross-chain totals, make separate requests per network and aggregate `volume_usd` values (after wei conversion). Do NOT sum `trader_count` across chains or time buckets.

---

## Response Example

```json
{
  "data": [
    {
      "timestamp": "2025-01-01T00:00:00Z",
      "volume_usd": "1500000000000000000000000",
      "trader_count": 4521,
      "tx_count": 12043
    }
  ]
}
```

Converted volume: `1500000000000000000000000 / 10^18 = 1,500,000 USD`

---

## Common Use Cases

1. **Daily volume report**: Use `period=1d`, sum `volume_usd` across hourly buckets (after wei conversion)
2. **Monthly growth**: Use `period=1m`, compare daily bucket values over time
3. **Multi-chain total volume**: Query each network separately, sum converted `volume_usd` values
4. **Active trader trend**: Track `trader_count` per bucket (do NOT sum across buckets for unique count)
