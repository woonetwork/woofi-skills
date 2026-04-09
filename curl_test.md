# WOOFi V2 API Curl Test Samples

This file contains readily executable `curl` commands for testing the WOOFi V2 API endpoints.

## 1. Get Quote (Base Chain — using chain name and token symbol)
**Scenario**: Swapping 0.1 Native ETH for USDC on Base.

```bash
curl -s -X POST "https://sapi.woofi.com/v2/quote" \
  -H "Content-Type: application/json" \
  -d '{
    "chain": "base",
    "sell_token": "ETH",
    "buy_token": "USDC",
    "sell_amount": "0.1"
  }' | jq .
```

---

## 2. Get Quote (Arbitrum — using chain name and token symbol)
**Scenario**: Swapping 1000 USDC for WBTC on Arbitrum.

```bash
curl -s -X POST "https://sapi.woofi.com/v2/quote" \
  -H "Content-Type: application/json" \
  -d '{
    "chain": "arbitrum",
    "sell_token": "USDC",
    "buy_token": "WBTC",
    "sell_amount": "1000"
  }' | jq .
```

---

## 3. Build Swap Transaction (Arbitrum — using token symbols)
**Scenario**: Generating the on-chain transaction payload to swap 1000 USDC for WBTC on Arbitrum, including approval checks.

```bash
curl -s -X POST "https://sapi.woofi.com/v2/swap" \
  -H "Content-Type: application/json" \
  -d '{
    "chain": "arbitrum",
    "sell_token": "USDC",
    "buy_token": "WBTC",
    "sell_amount": "1000",
    "to": "0x000000000000000000000000000000000000dEaD",
    "rebate_to": "0x000000000000000000000000000000000000dEaD",
    "signer_address": "0x000000000000000000000000000000000000dEaD"
  }' | jq .
```

---

## 4. Get Quote with Custom Slippage (Arbitrum)
**Scenario**: Swapping 1000 USDC for WBTC on Arbitrum, strictly enforcing a maximum slippage of 0.1% (default is 0.5%).

```bash
curl -s -X POST "https://sapi.woofi.com/v2/quote" \
  -H "Content-Type: application/json" \
  -d '{
    "chain": "arbitrum",
    "sell_token": "USDC",
    "buy_token": "WBTC",
    "sell_amount": "1000",
    "slippage_pct": 0.1
  }' | jq .
```

---

## 5. Build Swap Transaction (Base — Native ETH to USDC)
**Scenario**: Generating swap data to trade 0.1 Native ETH for USDC on Base, with a specific receiver address. Since it's native ETH, `needs_approve` will be false.

```bash
curl -s -X POST "https://sapi.woofi.com/v2/swap" \
  -H "Content-Type: application/json" \
  -d '{
    "chain": "base",
    "sell_token": "ETH",
    "buy_token": "USDC",
    "sell_amount": "0.1",
    "to": "0x1234567890123456789012345678901234567890",
    "rebate_to": "0x1234567890123456789012345678901234567890",
    "signer_address": "0x1234567890123456789012345678901234567890"
  }' | jq .
```

---

## 6. Build Swap Transaction (BSC — USDT to Native BNB)
**Scenario**: Swapping 500 USDT to Native BNB on Binance Smart Chain. This typically requires approving USDT first if the signer hasn't done so.

```bash
curl -s -X POST "https://sapi.woofi.com/v2/swap" \
  -H "Content-Type: application/json" \
  -d '{
    "chain": "bsc",
    "sell_token": "USDT",
    "buy_token": "BNB",
    "sell_amount": "500",
    "to": "0x1234567890123456789012345678901234567890",
    "rebate_to": "0x1234567890123456789012345678901234567890",
    "signer_address": "0x1234567890123456789012345678901234567890"
  }' | jq .
```

---

## 7. Build Swap Transaction with High Slippage (Polygon)
**Scenario**: Swapping 100 USDC to Native MATIC on Polygon with a high slippage tolerance of 2.5% during volatile market conditions.

```bash
curl -s -X POST "https://sapi.woofi.com/v2/swap" \
  -H "Content-Type: application/json" \
  -d '{
    "chain": "polygon",
    "sell_token": "USDC",
    "buy_token": "MATIC",
    "sell_amount": "100",
    "slippage_pct": 2.5,
    "to": "0x1234567890123456789012345678901234567890",
    "rebate_to": "0x1234567890123456789012345678901234567890",
    "signer_address": "0x1234567890123456789012345678901234567890"
  }' | jq .
```

---

## 8. Check Token Approval (Arbitrum)
**Scenario**: Check if USDC approval is needed before swapping on Arbitrum.

```bash
curl -s -X POST "https://sapi.woofi.com/v2/check_approval" \
  -H "Content-Type: application/json" \
  -d '{
    "chain": "arbitrum",
    "sell_token": "USDC",
    "owner": "0x000000000000000000000000000000000000dEaD",
    "sell_amount": "1000"
  }' | jq .
```

---

## 9. Cross-Chain Quote (Arbitrum → Base)
**Scenario**: Quote for bridging 100 USDC from Arbitrum to Base via Stargate.

```bash
curl -s -X POST "https://sapi.woofi.com/v2/cross_chain/quote" \
  -H "Content-Type: application/json" \
  -d '{
    "src_chain": "arbitrum",
    "dst_chain": "base",
    "src_token": "USDC",
    "dst_token": "USDC",
    "src_amount": "100"
  }' | jq .
```

---

## 10. Cross-Chain Swap (Arbitrum → Base)
**Scenario**: Build transaction to bridge 100 USDC from Arbitrum to Base.

```bash
curl -s -X POST "https://sapi.woofi.com/v2/cross_chain/swap" \
  -H "Content-Type: application/json" \
  -d '{
    "src_chain": "arbitrum",
    "dst_chain": "base",
    "src_token": "USDC",
    "dst_token": "USDC",
    "src_amount": "100",
    "to": "0x000000000000000000000000000000000000dEaD",
    "signer_address": "0x000000000000000000000000000000000000dEaD"
  }' | jq .
```

---

## 11. Quote Using Chain ID and Token Address (backward compatible)
**Scenario**: Same as #2 but using numeric chain ID and token addresses instead of names.

```bash
curl -s -X POST "https://sapi.woofi.com/v2/quote" \
  -H "Content-Type: application/json" \
  -d '{
    "chain": 42161,
    "sell_token": "0xaf88d065e77c8cC2239327C5EDb3A432268e5831",
    "buy_token": "0x2f2a2543b76a4166549f7aab2e75bef0aefc5b0f",
    "sell_amount": "1000"
  }' | jq .
```
