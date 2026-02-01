---
name: quantish-mcp-setup
description: Set up complete Quantish prediction market trading system (Discovery + Polymarket + Kalshi MCP servers)
user_invocable: true
---

# Quantish MCP Setup

Sets up 3 MCP servers for prediction market trading:
- **Discovery** - Search markets across Polymarket, Kalshi, Limitless
- **Polymarket** - Trade on Polymarket (crypto)
- **Kalshi** - Trade on Kalshi (CFTC-regulated via Solana)

## Instructions

### Step 1: Get User's Email

Ask for email using AskUserQuestion:
```
question: "What email should I use for your Quantish account?"
header: "Email"
options:
  - label: "Use my email"
    description: "I'll type my email address"
  - label: "Skip"
    description: "Use default account"
```

If "Skip", use `claude-user@quantish.live` as the email.

### Step 2: Get API Keys

Run these 4 curl commands, replacing USER_EMAIL with the email from Step 1:

**Polymarket key:**
```bash
curl -s -X POST https://quantish-sdk-production.up.railway.app/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"tools/call","params":{"name":"request_api_key","arguments":{"externalId":"USER_EMAIL"}},"id":1}'
```
Extract `apiKey` from `result.content[0].text` JSON. Format: `pk_live_...`

**Setup wallet** (use the key you just got):
```bash
curl -s -X POST https://quantish-sdk-production.up.railway.app/mcp \
  -H "Content-Type: application/json" \
  -H "x-api-key: POLYMARKET_KEY" \
  -d '{"jsonrpc":"2.0","method":"tools/call","params":{"name":"setup_wallet","arguments":{}},"id":1}'
```

**Kalshi key:**
```bash
curl -s -X POST https://kalshi-mcp-production-7c2c.up.railway.app/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"tools/call","params":{"name":"kalshi_signup","arguments":{"externalId":"USER_EMAIL"}},"id":1}'
```
Extract `apiKey` from `result.content[0].text` JSON.

**Discovery key:**
```bash
curl -s -X POST https://quantish.live/api/mcp/discovery-key \
  -H "Content-Type: application/json" \
  -d '{"name":"Claude Code - USER_EMAIL"}'
```
Extract `key` from response. Format: `qm_...`

### Step 3: Add MCP Servers

Run these 3 commands with the actual keys from Step 2:

```bash
claude mcp add --transport http quantish-discovery https://quantish.live/mcp --header "X-API-Key: DISCOVERY_KEY"
```

```bash
claude mcp add --transport http quantish https://quantish-sdk-production.up.railway.app/mcp --header "x-api-key: POLYMARKET_KEY"
```

```bash
claude mcp add --transport http quantish-kalshi https://kalshi-mcp-production-7c2c.up.railway.app/mcp --header "x-api-key: KALSHI_KEY"
```

### Step 4: Verify

```bash
claude mcp list
```

Should show all 3 quantish servers as connected.

### Step 5: Done

Tell user:
```
Setup complete! Restart Claude Code to load the servers.

After restart, try:
- "Search for Bitcoin markets"
- "Show my Polymarket balance"
- "Show my Kalshi positions"
```

## Next Steps

After setup, use these skills for trading:

- `/polymarket-trading` - Complete Polymarket trading guide
- `/kalshi-trading` - Complete Kalshi trading guide

## Notes
- If server already exists, `claude mcp add` will error - that's fine
- Keys are idempotent - same email returns same key
- Discovery has 5 keys/hour rate limit
