---
description: Query WOOFi cross-chain aggregated trading statistics including total volume, 24h stats, and per-network totals
triggers:
  - "total volume"
  - "total stats"
  - "cross chain stats"
  - "multi chain stats"
  - "aggregated stats"
  - "cumulative stats"
  - "24h stats"
  - "overall volume"
  - "all chains volume"
  - "yesterday volume"
---

# WOOFi Aggregated Statistics Skill

## Overview

This skill queries the WOOFi API aggregated statistics endpoints to retrieve cross-chain totals, per-network totals, and 24-hour cumulative statistics.

**Base URL**: `https://api.woofi.com`
**Authentication**: None required
**Rate Limit**: 5 requests/second

---

## Endpoints

### 1. GET /multi_total_stat

Returns aggregated statistics across **all** supported chains.

**Caching**: 5 seconds

**Parameters**: None

**Response Fields**:

| Field | Type | Description |
|-------|------|-------------|
| `past_24h_volume` | string | 24-hour trading volume in USD (18 decimals) |
| `total_volume_usd` | string | Total cumulative trading volume |
| `yesterday_volume_usd` | string | Trading volume from the previous UTC day |
| `last_week_volume_usd` | string | Trading volume for the last 7 days |
| `updated_at` | integer | Unix timestamp of the last update |

**Volume conversion**: Divide all volume values by `10^18` to get USD amounts.

---

### 2. GET /total_stat

Returns total volume, traders, and transactions for a **specific network**.

**Caching**: 60 seconds

**Parameters**:

| Parameter | Required | Values |
|-----------|----------|--------|
| `network` | Yes | See supported networks below |

**Response Fields**:

| Field | Type | Description |
|-------|------|-------------|
| `total_volume_usd` | string | Total trading volume in USD |
| `total_traders` | integer | Total unique traders |
| `total_txns` | integer | Total transactions |

---

### 3. GET /cumulate_stat

Returns **24-hour** cumulative statistics including turnover rate.

**Caching**: 5 seconds

**Parameters**:

| Parameter | Required | Values |
|-----------|----------|--------|
| `network` | Yes | See supported networks below |

**Response Fields**:

| Field | Type | Description |
|-------|------|-------------|
| `24h_volume_usd` | string | 24-hour volume |
| `24h_traders` | integer | 24-hour unique traders |
| `24h_txns` | integer | 24-hour transactions |
| `turnover_rate_percentage` | number | Turnover rate based on TVL |

---

## Supported Networks

**EVM**: `bsc`, `avax`, `fantom`, `polygon`, `arbitrum`, `optimism`, `zksync`, `polygon_zkevm`, `linea`, `base`, `mantle`, `sonic`, `berachain`, `hyperevm`, `monad`

**SVM**: `solana`

---

## Example Queries

### Cross-chain total volume
```
GET https://api.woofi.com/multi_total_stat
```

### Per-network lifetime stats
```
GET https://api.woofi.com/total_stat?network=arbitrum
```

### 24h cumulative stats on BSC
```
GET https://api.woofi.com/cumulate_stat?network=bsc
```

---

## Response Examples

### /multi_total_stat
```json
{
  "status": "ok",
  "data": {
    "past_24h_volume": "25000000000000000000000000",
    "total_volume_usd": "50000000000000000000000000000",
    "yesterday_volume_usd": "22000000000000000000000000",
    "last_week_volume_usd": "160000000000000000000000000",
    "updated_at": 1710000000
  }
}
```

Converted: `past_24h_volume = 25000000000000000000000000 / 10^18 = $25,000,000`

### /cumulate_stat
```json
{
  "status": "ok",
  "data": {
    "24h_volume_usd": "5000000000000000000000000",
    "24h_traders": 1200,
    "24h_txns": 3500,
    "turnover_rate_percentage": 45.2
  }
}
```

---

## Common Use Cases

1. **Dashboard headline stats**: Use `/multi_total_stat` for cross-chain total volume, 24h volume, and weekly volume
2. **Per-chain comparison**: Query `/total_stat` for each network to compare lifetime volumes
3. **Activity monitoring**: Use `/cumulate_stat` to track real-time 24h volume, traders, and turnover rate
4. **Yesterday recap**: Read `yesterday_volume_usd` from `/multi_total_stat` for daily reports
