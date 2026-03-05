---
description: Query WOOFi Swap trading volume broken down by integrator or aggregator source for a given period and network
triggers:
  - "volume by source"
  - "volume by integrator"
  - "traffic source"
  - "top integrators"
  - "who is sending volume"
  - "source breakdown"
  - "aggregator volume"
  - "woofi source stats"
---

# WOOFi Volume by Source Skill

## Overview

This skill queries the WOOFi Swap API `/source_stat` endpoint to retrieve trading volume broken down by integrator or aggregator source.

**Base URL**: `https://api.woofi.com`
**Endpoint**: `GET /source_stat`
**Authentication**: None required
**Rate Limit**: 120 requests/minute

---

## Parameters

| Parameter | Required | Values |
|-----------|----------|--------|
| `period`  | Yes | `1d`, `1w`, `1m`, `3m`, `1y`, `all` |
| `network` | Yes | See supported networks below |

### Supported Networks

`bsc`, `avalanche`, `polygon`, `arbitrum`, `optimism`, `linea`, `base`, `mantle`, `sonic`, `berachain`, `hyperevm`, `monad`, `solana`

---

## Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `source_id` | string | Unique identifier for the integrator/aggregator |
| `source_name` | string | Human-readable name of the traffic source |
| `volume_usd` | string | Volume in wei — divide by `10^18` to get USD value |
| `tx_count` | integer | Number of transactions from this source |
| `percentage` | number | Share of total volume as a percentage (0–100) |

**Note**: The response includes an `"Other"` catch-all category aggregating smaller sources not individually listed.

---

## Volume Conversion

All `volume_usd` values are in wei format. Convert with:
```
actual_usd_volume = volume_usd / 10^18
```

---

## Example Queries

### Top sources on Arbitrum for the past month
```
GET https://api.woofi.com/source_stat?period=1m&network=arbitrum
```

### All-time source breakdown on BSC
```
GET https://api.woofi.com/source_stat?period=all&network=bsc
```

### Cross-network integrator analysis
Make separate requests per network, then group results by `source_name` to aggregate a specific integrator's volume across chains.

---

## Response Example

```json
{
  "data": [
    {
      "source_id": "1inch",
      "source_name": "1inch",
      "volume_usd": "85000000000000000000000000",
      "tx_count": 45231,
      "percentage": 34.2
    },
    {
      "source_id": "other",
      "source_name": "Other",
      "volume_usd": "12000000000000000000000000",
      "tx_count": 6102,
      "percentage": 4.8
    }
  ]
}
```

---

## Common Use Cases

1. **Top integrator report**: Sort results by `percentage` descending to find the largest volume sources
2. **Specific integrator tracking**: Filter by `source_name` across multiple networks to total a partner's contribution
3. **Market share trends**: Compare `percentage` values across different `period` values to identify growing or declining sources
4. **Partner analytics**: Use `tx_count` alongside `volume_usd` to understand average trade size per integrator
