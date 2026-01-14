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

## Usage with Claude Desktop

Add the following to your Claude Desktop configuration (`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS):

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

See the `examples/` directory for reference implementations.
