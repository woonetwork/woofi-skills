---
description: Query WOOFi Pro (DEX) perpetual trading daily volume data
triggers:
  - "perps volume"
  - "perpetual volume"
  - "woofi pro"
  - "woofi dex"
  - "perp trading"
  - "daily perp volume"
  - "futures volume"
  - "orderly volume"
---

# WOOFi Pro Perps Volume Skill

## Overview

This skill queries the WOOFi API `/woofi_pro/perps_volume` endpoint to retrieve daily perpetual trading volume for the WOOFi Pro broker.

**Base URL**: `https://api.woofi.com`
**Endpoint**: `GET /woofi_pro/perps_volume`
**Authentication**: None required
**Rate Limit**: 5 requests/second
**Caching**: 5 seconds

---

## Parameters

None.

---

## Response Fields

Returns a list of daily volume data points for the last 80 days.

**Data source**: Fetched from the Orderly Network API (`OrderlyApi.fetch_volume_broker_daily()`).

---

## Example Query

### Get WOOFi Pro daily perps volume
```
GET https://api.woofi.com/woofi_pro/perps_volume
```

---

## Response Example

```json
{
  "status": "ok",
  "data": [
    {
      "date": "2025-01-15",
      "volume": 12500000.50
    },
    {
      "date": "2025-01-14",
      "volume": 9800000.25
    }
  ]
}
```

---

## Common Use Cases

1. **Perps trading trends**: Analyze daily volume trends over the last 80 days
2. **Product comparison**: Compare perps volume against swap volume from `/multi_total_stat`
3. **Growth tracking**: Monitor WOOFi Pro adoption by tracking volume trajectory
4. **Broker performance**: Assess WOOFi's share of Orderly Network volume
