---
name: polymarket-trading
version: 1.0.0
description: Trade prediction markets on Polymarket via Quantish MCP
author: Quantish
tags: [trading, polymarket, prediction-markets, crypto]
---

# Polymarket Trading Skill

Complete guide to trading on Polymarket through the Quantish MCP server.

## Prerequisites

Run `/quantish-mcp-setup` first to connect to the MCP servers.

## Trading Workflow

### 1. Check Wallet Status

```tool
mcp__quantish__get_wallet_status
```

**Response fields:**
- `safeAddress`: Your trading wallet (Polygon)
- `safeDeployed`: Must be `true`
- `approvals`: All should be `true` (usdc, ctf, negRisk)
- `isReady`: Must be `true` to trade

### 2. Check Balances

```tool
mcp__quantish__get_balances
```

**Response:**
```json
{
  "safe": {
    "address": "0x...",
    "usdc": "10.0",      // Available to trade
    "nativeUsdc": "0.0"  // Circle USDC (different token)
  }
}
```

### 3. Find Markets

Use the discovery server to search:

```tool
mcp__quantish-discovery__search_markets
  query: "government shutdown"
  limit: 5
```

**Key fields from response:**
- `conditionId`: Market identifier (0x...)
- `tokenId`: Outcome token ID (long number string)
- `prices[].outcome`: "Yes" or "No"
- `prices[].price`: Current price (0.01-0.99)

### 4. Check Orderbook (IMPORTANT - DO THIS FIRST)

```tool
mcp__quantish__get_orderbook
  tokenId: "8008742846391096366381429238657491392199916771102623264605530168655505543184"
```

**Response shows:**
- `bids`: Buy orders (sorted low to high)
- `asks`: Sell orders (sorted high to low)
- Best bid = highest bid price
- Best ask = lowest ask price

**ALWAYS check orderbook before placing orders to get best price.**

### 5. Get Current Price

```tool
mcp__quantish__get_price
  tokenId: "8008742846391096366381429238657491392199916771102623264605530168655505543184"
```

Returns `midpointPrice` between best bid/ask.

### 6. Place Order

```tool
mcp__quantish__place_order
  conditionId: "0xd5a91c9ee50ba80385283714f2a66b1e16d544a682af2af06a8f57fcf1d0233d"
  tokenId: "8008742846391096366381429238657491392199916771102623264605530168655505543184"
  side: "BUY"
  price: 0.45
  size: 4
```

**Parameters:**
| Param | Type | Description |
|-------|------|-------------|
| conditionId | string | Market ID (from search) |
| tokenId | string | YES or NO token ID |
| side | "BUY" or "SELL" | Direction |
| price | number | Limit price 0.01-0.99 |
| size | number | Number of shares |
| orderType | string | "GTC" (default), "FOK", "FAK", "GTD" |

**Order Types:**
- `GTC`: Good Till Cancelled - rests on book, crosses spread if price allows
- `FOK`: Fill Or Kill - must fill entirely or fails
- `FAK`: Fill And Kill - fills what it can, cancels rest
- `GTD`: Good Till Date - requires `expiration` timestamp

**Cost calculation:** `price × size = USDC cost`

**Response:**
```json
{
  "success": true,
  "orderId": "f3a665c9-...",
  "status": "LIVE",  // or "FILLED"
  "message": "Order is live: BUY 4 shares at $0.45"
}
```

### 7. Check Positions

```tool
mcp__quantish__get_positions
```

**Response:**
```json
{
  "positions": [{
    "outcome": "Yes",
    "size": 4,
    "avgPrice": 0.45,
    "currentPrice": 0.43,
    "currentValue": 1.72,
    "pnl": -0.08,
    "marketTitle": "Will the government shutdown last 5 days or more?"
  }]
}
```

### 8. Check Orders

```tool
mcp__quantish__get_orders
  status: "LIVE"  // optional: PENDING, LIVE, FILLED, CANCELLED, FAILED
```

### 9. Cancel Order

```tool
mcp__quantish__cancel_order
  orderId: "f3a665c9-..."
```

### 10. Sync Order Status

```tool
mcp__quantish__sync_order_status
  orderId: "f3a665c9-..."
```

Note: May show `sizeMatched: 0` due to API lag even if filled. Use `get_positions` to confirm.

### 11. Claim Winnings (After Market Resolves)

```tool
mcp__quantish__get_claimable_winnings
```

```tool
mcp__quantish__claim_winnings
  positionId: "..."  // optional, claims all if omitted
```

## Advanced Operations

### Merge Tokens (Exit Position at $1)

If you hold both YES and NO tokens, merge them back to USDC:

```tool
mcp__quantish__merge_tokens
  conditionId: "0x..."
```

### Transfer USDC

```tool
mcp__quantish__transfer_usdc
  toAddress: "0x..."
  amount: 5.0
```

### Get Deposit Addresses

```tool
mcp__quantish__get_deposit_addresses
```

Supports deposits from Ethereum, Polygon, Arbitrum, Optimism, Base, Solana, and Bitcoin.

## Best Practices

1. **Always check orderbook first** - Know the spread before placing orders
2. **Use limit orders** - GTC orders get price improvement
3. **Start small** - $1-2 trades until comfortable
4. **Verify fills** - Use `get_positions` not just order status
5. **Check balances** - Ensure sufficient USDC before trading

## Error Handling

| Error | Cause | Fix |
|-------|-------|-----|
| "not enough balance" | Insufficient USDC | Deposit more or reduce size |
| "order failed" | Price moved | Check orderbook, adjust price |
| "invalid token" | Wrong tokenId | Get fresh market data |

## Example: Complete Trade Flow

```
1. get_wallet_status → Confirm ready
2. get_balances → Check USDC
3. search_markets → Find market
4. get_orderbook → See bid/ask spread
5. place_order → Execute trade
6. get_positions → Verify fill
```
