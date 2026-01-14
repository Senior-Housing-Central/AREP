# Getting Started with AREP

This guide will help you get started with the AI Real Estate Protocol (AREP), whether you're building a client application, implementing a provider, or integrating with AI assistants.

## What is AREP?

AREP is an open standard that enables AI agents to:

- **Discover** properties for sale or rent
- **Get valuations** and market data
- **Submit applications** (rentals) or **offers** (purchases)
- **Complete transactions** through leases or contracts

## Quick Start

### 1. Find an AREP Provider

AREP providers publish a discovery manifest at `/.well-known/arep`:

```bash
curl https://api.example.com/.well-known/arep
```

```json
{
  "arep": {
    "version": "2026-01",
    "provider": {
      "name": "Example Realty",
      "type": "marketplace",
      "transaction_types": ["rental", "sale"]
    },
    "capabilities": {
      "discovery": { "version": "2026-01" },
      "pricing": { "version": "2026-01" },
      "offer": { "version": "2026-01" }
    },
    "transports": {
      "rest": {
        "endpoint": "https://api.example.com/arep/v1"
      }
    }
  }
}
```

### 2. Search for Properties

#### For Sale

```bash
curl "https://api.example.com/arep/v1/properties?transaction_type=for_sale&city=Austin&state=TX&price_max=500000&bedrooms_min=3"
```

#### For Rent

```bash
curl "https://api.example.com/arep/v1/properties?transaction_type=for_rent&city=Seattle&state=WA&price_max=2500"
```

### 3. Get Property Details

```bash
curl "https://api.example.com/arep/v1/properties/arep:example-123456"
```

### 4. Get Valuations

```bash
# Property value estimate
curl "https://api.example.com/arep/v1/estimate/arep:example-123456"

# Comparable sales
curl "https://api.example.com/arep/v1/comparables/arep:example-123456?transaction_type=sold&radius_miles=1"

# Market trends
curl "https://api.example.com/arep/v1/market/Austin,TX?period=12_months"
```

### 5. Submit an Offer

#### Rental Application

```bash
curl -X POST "https://api.example.com/arep/v1/applications" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{
    "property_id": "arep:example-123456",
    "applicant": {
      "name": { "first": "John", "last": "Smith" },
      "email": "john@example.com",
      "phone": "+1-555-123-4567"
    },
    "desired_move_in": "2026-03-01",
    "desired_lease_term": 12
  }'
```

#### Purchase Offer

```bash
curl -X POST "https://api.example.com/arep/v1/offers" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{
    "property_id": "arep:example-123456",
    "buyer": {
      "name": "John Smith",
      "email": "john@example.com"
    },
    "offer_price": 475000,
    "earnest_money": 10000,
    "financing": {
      "type": "conventional",
      "pre_approved": true
    },
    "contingencies": {
      "inspection": { "included": true, "days": 10 },
      "appraisal": { "included": true },
      "financing": { "included": true, "days": 21 }
    },
    "closing_date": "2026-03-15"
  }'
```

## Authentication

AREP supports multiple authentication methods:

### OAuth 2.0 (Recommended)

```bash
# Client credentials flow
curl -X POST "https://api.example.com/oauth/token" \
  -d "grant_type=client_credentials" \
  -d "client_id=YOUR_CLIENT_ID" \
  -d "client_secret=YOUR_CLIENT_SECRET" \
  -d "scope=property:read application:write"
```

### API Key

```bash
curl -H "X-API-Key: YOUR_API_KEY" \
  "https://api.example.com/arep/v1/properties?city=Austin"
```

## Scopes

| Scope | Description |
|-------|-------------|
| `property:read` | Search and view properties |
| `property:write` | Create/update listings |
| `estimate:read` | Get valuations and comps |
| `application:read` | View applications |
| `application:write` | Create/submit applications |
| `offer:read` | View purchase offers |
| `offer:write` | Create/submit offers |
| `transaction:read` | View documents |
| `transaction:sign` | Sign documents |

## Using with AI Assistants

### Claude Desktop (MCP)

Add to your Claude Desktop config:

```json
{
  "mcpServers": {
    "arep": {
      "command": "node",
      "args": ["/path/to/arep-mcp-server/dist/server.js"],
      "env": {
        "AREP_ENDPOINT": "https://api.example.com/arep/v1",
        "AREP_API_KEY": "your-api-key"
      }
    }
  }
}
```

Then ask Claude:

> "Find me 3-bedroom homes for sale in Austin under $500k"

### A2A (Agent-to-Agent)

Discover agent capabilities at `/.well-known/agent.json`:

```bash
curl "https://api.example.com/.well-known/agent.json"
```

## Next Steps

- Read the [full specification](AREP_Specification.md)
- Explore the [architecture](architecture.md)
- Check out the [OpenAPI spec](../openapi/arep-v1.yaml)
- Try the [MCP tools](../mcp/tools.json)

## Support

- **Specification**: [GitHub](https://github.com/arep-dev/spec)
- **Email**: spec@arep.dev
