# AREP MCP Integration

This directory contains MCP (Model Context Protocol) tool definitions for the AI Real Estate Protocol.

## Overview

MCP enables AI assistants like Claude to interact with AREP providers through a standardized tool interface. The tools defined here map directly to AREP capabilities.

## Tools

| Tool | Description |
|------|-------------|
| `search_properties` | Search for properties to buy or rent |
| `get_property_details` | Get detailed property information |
| `get_estimate` | Get property value or rent estimates |
| `get_comparables` | Find comparable properties for valuation |
| `get_market_trends` | Get market statistics for an area |
| `calculate_affordability` | Calculate home buying affordability |
| `create_offer` | Create rental application or purchase offer |
| `get_offer_status` | Check offer/application status |

---

## Reference Implementation: Senior Housing Central

[Senior Housing Central (SHC)](https://seniorhousingcentral.com) provides a production reference implementation of AREP with MCP support for senior housing rentals.

### Endpoints

| Environment | MCP Endpoint | REST Endpoint |
|-------------|--------------|---------------|
| **Production** | `https://api.seniorhousingcentral.com/arep/mcp/sse` | `https://api.seniorhousingcentral.com/arep/v1` |
| **Staging** | `https://api-stage.seniorhousingcentral.com/arep/mcp/sse` | `https://api-stage.seniorhousingcentral.com/arep/v1` |
| **Development** | `https://api-dev.seniorhousingcentral.com/arep/mcp/sse` | `https://api-dev.seniorhousingcentral.com/arep/v1` |

### Available Tools (Discovery Capability)

SHC implements the **Discovery** capability for senior housing rentals:

| Tool | Description |
|------|-------------|
| `search_properties` | Search senior housing rentals by location, price, and features |
| `get_property_details` | Get detailed property information including amenities and walk scores |
| `get_similar_properties` | Find properties similar to a given property |

### Claude Desktop Configuration

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "shc-arep": {
      "url": "https://api.seniorhousingcentral.com/arep/mcp/sse"
    }
  }
}
```

### Example Conversation

**User:** Find senior housing in Los Angeles under $2500/month

**Claude:** I'll search for senior housing in LA.

*Uses `search_properties` tool:*
```json
{
  "city": "Los Angeles",
  "state": "CA",
  "price_max": 2500
}
```

**Response:**
```json
{
  "total": 47,
  "properties": [
    {
      "property_id": "arep:shc-3a5b5f62",
      "address": { "street": "123 Senior Living Way", "city": "Los Angeles", "state": "CA" },
      "price": { "rent_min": 1800, "rent_max": 2200 },
      "features": { "beds_min": 1, "beds_max": 2 }
    }
  ]
}
```

### Health Check

```bash
curl https://api.seniorhousingcentral.com/arep/mcp/health
```

Response:
```json
{
  "status": "ok",
  "server": "shc-arep",
  "version": "1.0.0",
  "tools": ["search_properties", "get_property_details", "get_similar_properties"]
}
```

---

## Generic Usage with Claude Desktop

For other AREP providers using stdio transport:

```json
{
  "mcpServers": {
    "arep": {
      "command": "node",
      "args": ["/path/to/arep-mcp-server/dist/server.js"],
      "env": {
        "AREP_ENDPOINT": "https://api.provider.com/arep/v1",
        "AREP_API_KEY": "your-api-key"
      }
    }
  }
}
```

## Example Conversations

### Property Search

**User:** Find me 3-bedroom homes for sale in Austin under $500k

**Claude:** I'll search for homes matching your criteria.

*Uses `search_properties` tool with:*
```json
{
  "transaction_type": "for_sale",
  "location": { "city": "Austin", "state": "TX" },
  "price_max": 500000,
  "bedrooms_min": 3
}
```

### Property Valuation

**User:** What's my house at 123 Main St worth?

**Claude:** Let me get an estimate for that property.

*Uses `get_estimate` tool with:*
```json
{
  "property_id": "arep:address-123-main-st",
  "estimate_type": "both"
}
```

### Affordability

**User:** How much house can I afford with a $120k salary?

**Claude:** I'll calculate your affordability.

*Uses `calculate_affordability` tool with:*
```json
{
  "annual_income": 120000,
  "down_payment_percent": 20,
  "loan_term_years": 30
}
```

## MCP JSON-RPC Format

Tools are called using the standard MCP JSON-RPC format:

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "search_properties",
    "arguments": {
      "transaction_type": "for_sale",
      "location": { "city": "Austin", "state": "TX" },
      "price_max": 500000
    }
  },
  "id": 1
}
```

## Implementation

To implement an AREP MCP server, you need to:

1. Create an MCP server using `@modelcontextprotocol/sdk`
2. Register the tools from `tools.json`
3. Map tool calls to the AREP REST API endpoints

### Transport Options

| Transport | Use Case | Example |
|-----------|----------|---------|
| **SSE** | Deployed API (recommended) | SHC uses this for production |
| **Stdio** | Local/development | CLI-based MCP servers |

### Reference Implementations

| Provider | Source | Capabilities |
|----------|--------|--------------|
| Senior Housing Central | [github.com/Senior-Housing-Central/shc](https://github.com/Senior-Housing-Central/shc) | Discovery (rentals) |

### SHC Implementation Details

The SHC MCP server (`apps/backend/src/mcp/`):

```
src/mcp/
├── sse-server.ts   # Express router with SSE transport
├── tools.ts        # Tool definitions using Zod schemas
├── handlers.ts     # Tool execution handlers
└── index.ts        # Standalone stdio server (for local testing)
```

Key files:
- **SSE Transport**: Uses `@modelcontextprotocol/sdk/server/sse.js`
- **Tool Schemas**: Defined with Zod, converted to JSON Schema
- **Handlers**: Query Hasura GraphQL, map to AREP format
