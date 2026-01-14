# AI Real Estate Protocol (AREP)

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.0.0--draft-orange.svg)](CHANGELOG.md)
[![Spec](https://img.shields.io/badge/spec-2026--01-green.svg)](docs/AREP_Specification.md)

**AREP** is an open standard enabling AI agents to discover properties, get valuations, submit applications or offers, and complete real estate transactions. AREP supports both **rental** and **sale** transactions through a unified protocol.

## Features

| Capability | Rentals | Sales | Description |
|------------|---------|-------|-------------|
| **Discovery** | ✓ | ✓ | Property search, details, availability |
| **Pricing** | ✓ | ✓ | Rent estimates, valuations, comps, market data |
| **Offer** | Application | Purchase Offer | Submit rental application or purchase offer |
| **Transaction** | Lease | Contract/Closing | Complete the transaction |

## Transport Support

AREP is transport-agnostic and works over:

- **REST API** - Standard HTTP/JSON endpoints
- **MCP** - Model Context Protocol for AI assistants
- **A2A** - Agent-to-Agent communication

## Quick Start

### 1. Discover an AREP Provider

```bash
curl https://api.example.com/.well-known/arep
```

```json
{
  "arep": {
    "version": "2026-01",
    "provider": {
      "name": "Example Realty",
      "transaction_types": ["rental", "sale"]
    },
    "capabilities": {
      "discovery": { "version": "2026-01" },
      "pricing": { "version": "2026-01" },
      "offer": { "version": "2026-01" }
    }
  }
}
```

### 2. Search Properties

```bash
curl "https://api.example.com/arep/v1/properties?transaction_type=for_sale&city=Austin&state=TX&price_max=500000"
```

### 3. Get Property Details

```bash
curl "https://api.example.com/arep/v1/properties/arep:zillow-123456"
```

### 4. Submit an Offer

```bash
curl -X POST "https://api.example.com/arep/v1/offers" \
  -H "Content-Type: application/json" \
  -d '{
    "property_id": "arep:zillow-123456",
    "offer_type": "purchase_offer",
    "offer_price": 475000,
    "buyer": {
      "name": "John Smith",
      "email": "john@example.com"
    }
  }'
```

## Documentation

- [Specification](docs/AREP_Specification.md) - Full protocol specification
- [Getting Started](docs/getting-started.md) - Quick start guide
- [Architecture](docs/architecture.md) - Architecture deep dive

## Schemas

JSON Schemas for validation:

- [`schemas/property.json`](schemas/property.json) - Property listing schema
- [`schemas/manifest.json`](schemas/manifest.json) - Discovery manifest schema
- [`schemas/application.json`](schemas/application.json) - Rental application schema
- [`schemas/offer.json`](schemas/offer.json) - Purchase offer schema
- [`schemas/agent-card.json`](schemas/agent-card.json) - A2A Agent Card schema

## API Specification

- [OpenAPI 3.0](openapi/arep-v1.yaml) - REST API specification

## MCP Integration

AREP provides MCP tool definitions for AI assistants:

- [`mcp/tools.json`](mcp/tools.json) - MCP tool definitions

### Claude Desktop Configuration

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

## Extensions

AREP supports vertical-specific extensions:

| Namespace | Purpose |
|-----------|---------|
| `org.arep.screening` | Tenant/buyer screening |
| `org.arep.mortgage` | Mortgage pre-approval, rates |
| `org.arep.senior` | Senior housing (care levels) |
| `org.arep.commercial` | Commercial real estate |
| `gov.hud.affordability` | HUD programs (Section 8, FHA) |

## RESO Alignment

AREP aligns with [RESO Data Dictionary 2.0](https://www.reso.org/data-dictionary/) for industry compatibility.

## Contributing

We welcome contributions! Please see our contributing guidelines (coming soon).

## License

Apache 2.0 - See [LICENSE](LICENSE) for details.

## Links

- **Specification**: [docs/AREP_Specification.md](docs/AREP_Specification.md)
- **Repository**: https://github.com/arep-dev/spec
- **Contact**: spec@arep.dev
