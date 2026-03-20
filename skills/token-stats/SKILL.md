---
description: Query WOOFi per-token 24-hour trading statistics including TVL, volume, and turnover rate
triggers:
  - "token stats"
  - "token volume"
  - "token tvl"
  - "token trading"
  - "per token stats"
  - "token turnover"
  - "token 24h"
  - "which token"
  - "top tokens"
---

# WOOFi Token Statistics Skill

## Overview

This skill queries the WOOFi API `/token_stat` endpoint to retrieve 24-hour statistics for each token supported by WOOFi on a given network.

**Base URL**: `https://api.woofi.com`
**Endpoint**: `GET /token_stat`
**Authentication**: None required
**Rate Limit**: 5 requests/second
**Caching**: 5 seconds

---

## Parameters

| Parameter | Required | Values |
|-----------|----------|--------|
| `network` | Yes | See supported networks below |

### Supported Networks

**EVM**: `bsc`, `avax`, `fantom`, `polygon`, `arbitrum`, `optimism`, `zksync`, `polygon_zkevm`, `linea`, `base`, `mantle`, `sonic`, `berachain`, `hyperevm`, `monad`

**SVM**: `solana`

---

## Response Fields

Each token object contains:

| Field | Type | Description |
|-------|------|-------------|
| `logo_url` | string | URL to the token logo image |
| `symbol` | string | Token symbol (e.g., ETH, USDC) |
| `decimals` | integer | Token decimal places |
| `tvl` | string | Total Value Locked (18 decimals) |
| `24h_volume_usd` | string | 24-hour volume in USD (18 decimals) |
| `24h_txns` | integer | 24-hour transaction count |
| `turnover_rate_percentage` | number | Turnover rate: volume / TVL |

---

## Value Conversion

Both `tvl` and `24h_volume_usd` are strings with 18 decimals. Convert with:
```
actual_value = value / 10^18
```

---

## Example Query

### Token stats on Arbitrum
```
GET https://api.woofi.com/token_stat?network=arbitrum
```

---

## Response Example

```json
{
  "status": "ok",
  "data": [
    {
      "logo_url": "https://example.com/eth.png",
      "symbol": "ETH",
      "decimals": 18,
      "tvl": "8000000000000000000000000",
      "24h_volume_usd": "3500000000000000000000000",
      "24h_txns": 2100,
      "turnover_rate_percentage": 43.75
    },
    {
      "logo_url": "https://example.com/usdc.png",
      "symbol": "USDC",
      "decimals": 6,
      "tvl": "5000000000000000000000000",
      "24h_volume_usd": "2000000000000000000000000",
      "24h_txns": 1800,
      "turnover_rate_percentage": 40.0
    }
  ]
}
```

Converted ETH TVL: `8000000000000000000000000 / 10^18 = $8,000,000`

---

## Common Use Cases

1. **Token ranking**: Sort by `24h_volume_usd` to find the most actively traded tokens on a network
2. **Liquidity depth**: Compare `tvl` across tokens to assess pool depth
3. **Turnover analysis**: High `turnover_rate_percentage` indicates efficient capital utilization
4. **Cross-chain token comparison**: Query multiple networks and filter by `symbol` to compare a token's stats across chains
5. **Activity monitoring**: Track `24h_txns` to identify trending tokens
