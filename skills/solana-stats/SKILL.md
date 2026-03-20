---
description: Query WOOFi Solana-specific pool statistics from Redis
triggers:
  - "solana stats"
  - "solana pool"
  - "solana stat"
  - "solana trading"
  - "solana volume"
---

# WOOFi Solana Statistics Skill

## Overview

This skill queries the WOOFi API `/solana_stat` endpoint to retrieve raw Solana pool statistics.

**Base URL**: `https://api.woofi.com`
**Endpoint**: `GET /solana_stat`
**Authentication**: None required
**Rate Limit**: 5 requests/second
**Caching**: 5 seconds

---

## Parameters

| Parameter | Required | Values |
|-----------|----------|--------|
| `filter_days` | No | Number of days to look back. Default: `14` |

---

## Example Queries

### Solana pool stats (default 14 days)
```
GET https://api.woofi.com/solana_stat
```

### Solana pool stats (last 7 days)
```
GET https://api.woofi.com/solana_stat?filter_days=7
```

---

## Notes

- This is a specialized endpoint returning raw Solana pool statistics directly from Redis
- For general Solana trading stats (volume, traders, txns), use the `/stat?network=solana` endpoint instead (covered by the `trading-stats` skill)
- Use `filter_days` to control the lookback window

---

## Common Use Cases

1. **Solana pool monitoring**: Track Solana-specific pool metrics over a configurable time window
2. **Solana health check**: Inspect raw pool data for anomalies or issues
3. **Custom analysis**: Use a shorter `filter_days` for recent trends or longer for broader analysis
