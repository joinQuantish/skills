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

**Setup Polymarket trading wallet (REQUIRED — use the key you just got):**

The Polymarket API key comes with an EOA address, but that is NOT the trading wallet. You MUST call `setup_wallet` to create the Safe wallet that is actually used to trade on Polymarket. Without this call, the user cannot place orders.

```bash
curl -s -X POST https://quantish-sdk-production.up.railway.app/mcp \
  -H "Content-Type: application/json" \
  -H "x-api-key: POLYMARKET_KEY" \
  -d '{"jsonrpc":"2.0","method":"tools/call","params":{"name":"setup_wallet","arguments":{}},"id":1}'
```
Extract the `safeAddress` from the response — this is the wallet the user should send USDC to for trading.

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

### Step 3: Add All 3 MCP Servers to Claude Code (CRITICAL)

**Getting API keys is NOT enough.** You MUST run these commands to register each server in your Claude Code configuration. Without this step, none of the MCP tools will be available. Each command should print "Added HTTP MCP server ... to local config" on success.

```bash
claude mcp add --transport http quantish-discovery https://quantish.live/mcp --header "X-API-Key: DISCOVERY_KEY"
```

```bash
claude mcp add --transport http quantish https://quantish-sdk-production.up.railway.app/mcp --header "x-api-key: POLYMARKET_KEY"
```

```bash
claude mcp add --transport http quantish-kalshi https://kalshi-mcp-production-7c2c.up.railway.app/mcp --header "x-api-key: KALSHI_KEY"
```

Verify each command printed "Added HTTP MCP server" with exit code 0. If it says "already exists", that server is already configured — that's fine.

### Step 4: Tell User to Restart Claude Code (REQUIRED)

MCP servers do NOT become available until Claude Code is restarted. The tools will not appear in the current session. Tell the user:

```
All 3 MCP servers are configured! You must exit and restart Claude Code for the tools to load.

After restarting, try:
- "Search for Bitcoin markets"
- "Show my Polymarket balance"
- "Show my Kalshi positions"
```

**Do not skip this step.** If the user asks why tools aren't showing up, the answer is always: restart Claude Code.

## Notes
- If server already exists, `claude mcp add` returns exit code 1 with "already exists" - that's fine, it's already configured
- Keys are idempotent - same email returns same key
- Discovery has 5 keys/hour rate limit
- All 3 servers must be added via `claude mcp add` — the API keys alone are useless without this configuration step
