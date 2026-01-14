# AREP Architecture

This document describes the architecture of the AI Real Estate Protocol (AREP).

## Overview

AREP is a layered protocol designed to be transport-agnostic, extensible, and incrementally adoptable.

```
┌─────────────────────────────────────────────────────────────────┐
│                        EXTENSIONS                                │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐           │
│  │Screening │ │ Mortgage │ │  Senior  │ │Commercial│  ...      │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘           │
├─────────────────────────────────────────────────────────────────┤
│                       CAPABILITIES                               │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐           │
│  │Discovery │ │ Pricing  │ │  Offer   │ │Transaction│          │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘           │
├─────────────────────────────────────────────────────────────────┤
│                       CORE SERVICE                               │
│            Protocol negotiation, auth, transport                 │
└─────────────────────────────────────────────────────────────────┘
```

## Design Principles

### 1. Unified Model

AREP uses a single protocol for both **rentals** and **sales**. The `transaction_type` field distinguishes between them:

```json
{
  "property_id": "arep:example-123",
  "transaction_type": "for_sale",  // or "for_rent"
  "property_type": "single_family",
  "status": "active"
}
```

### 2. Transport Agnostic

AREP works over multiple transports:

| Transport | Use Case | Format |
|-----------|----------|--------|
| **REST** | Traditional API integration | HTTP/JSON |
| **MCP** | AI assistant tools | JSON-RPC 2.0 |
| **A2A** | Agent-to-agent communication | JSON-RPC 2.0 |
| **GraphQL** | Flexible queries | GraphQL |

### 3. Incrementally Adoptable

Providers implement only the capabilities they need:

- **Minimal**: Discovery only (search/browse)
- **Standard**: Discovery + Pricing
- **Full**: All capabilities including Offer and Transaction

### 4. Extension-Friendly

Vertical-specific functionality lives in extension namespaces:

```json
{
  "extensions": {
    "org.arep.screening": { "version": "2026-01" },
    "org.arep.mortgage": { "version": "2026-01" },
    "gov.hud.affordability": { "version": "2026-01" }
  }
}
```

## Core Capabilities

### Discovery

Property search and retrieval.

```
┌─────────────┐     search      ┌─────────────┐
│   Client    │────────────────▶│   Provider  │
│             │◀────────────────│             │
└─────────────┘    properties   └─────────────┘
```

**Operations:**
- `search` - Search by criteria
- `get` - Get property by ID
- `similar` - Find similar properties
- `history` - Get price/transaction history

### Pricing

Valuations, estimates, and market data.

```
┌─────────────┐    estimate     ┌─────────────┐
│   Client    │────────────────▶│   Provider  │
│             │◀────────────────│   (+ Data)  │
└─────────────┘   value/comps   └─────────────┘
```

**Operations:**
- `estimate` - Property value estimate
- `rent_estimate` - Rental price estimate
- `comparables` - Similar sold/rented properties
- `market_trends` - Area market statistics
- `affordability` - Calculate buying power

### Offer

Submit rental applications or purchase offers.

```
┌─────────────┐    create       ┌─────────────┐     notify     ┌─────────────┐
│   Buyer/    │────────────────▶│   Provider  │───────────────▶│   Seller/   │
│   Renter    │◀────────────────│             │◀───────────────│   Landlord  │
└─────────────┘    status       └─────────────┘    response    └─────────────┘
```

**Rental Application Flow:**
1. `create_application` → draft
2. `update_application` → add details
3. `submit_application` → submitted
4. Provider reviews → approved/denied

**Purchase Offer Flow:**
1. `create_offer` → draft
2. `submit_offer` → submitted
3. Seller responds → accepted/countered/rejected
4. Negotiation → accepted
5. `under_contract` → closing

### Transaction

Complete leases or purchase contracts.

**Rental:**
```
approved_application → create_lease → sign_lease → active_lease
```

**Sale:**
```
accepted_offer → create_contract → sign_contract → closing → closed
```

## Data Model

### Property ID Format

```
arep:{source}-{id}

Examples:
  arep:zillow-123456
  arep:mls-abc-789
  arep:provider-unit-42
```

### Core Schemas

| Schema | Description |
|--------|-------------|
| `property.json` | Unified property listing |
| `application.json` | Rental application |
| `offer.json` | Purchase offer |
| `manifest.json` | Provider discovery |
| `agent-card.json` | A2A agent definition |

### RESO Alignment

AREP aligns with RESO Data Dictionary 2.0:

| AREP | RESO |
|------|------|
| `property_id` | `ListingKey` |
| `transaction_type` | `PropertySubType` |
| `pricing.list_price` | `ListPrice` |
| `details.bedrooms` | `BedroomsTotal` |
| `details.sqft` | `LivingArea` |

## Transport Bindings

### REST API

```
Base: https://api.provider.com/arep/v1

GET  /properties              Search
GET  /properties/{id}         Get details
GET  /estimate/{id}           Get estimate
GET  /comparables/{id}        Get comps
GET  /market/{location}       Market trends
POST /affordability           Calculate
POST /applications            Create rental app
POST /offers                  Create purchase offer
```

### MCP (Model Context Protocol)

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "search_properties",
    "arguments": {
      "transaction_type": "for_sale",
      "location": { "city": "Austin", "state": "TX" }
    }
  },
  "id": 1
}
```

### A2A (Agent-to-Agent)

```json
{
  "jsonrpc": "2.0",
  "method": "message/send",
  "params": {
    "message": {
      "role": "user",
      "parts": [{
        "type": "text",
        "text": "Find homes for sale in Austin under $500k"
      }]
    }
  },
  "id": "task-123"
}
```

## Security

### Authentication

| Method | Use Case |
|--------|----------|
| OAuth 2.0 Client Credentials | B2B API access |
| OAuth 2.0 Authorization Code | End-user apps |
| API Keys | Development/testing |

### Authorization Scopes

```
property:read       Search and view
property:write      Create/update listings
estimate:read       Valuations and comps
application:read    View applications
application:write   Submit applications
offer:read          View offers
offer:write         Submit offers
transaction:read    View documents
transaction:sign    E-sign documents
```

### Data Privacy

- Minimal disclosure by default
- Consent tracking for PII
- CCPA/GDPR compliance
- Elevated consent for sensitive data (SSN, financials)

## Extension Architecture

Extensions add vertical-specific functionality without modifying core capabilities.

### Namespace Convention

```
{org}.{domain}.{extension}

Examples:
  org.arep.screening      Official screening extension
  org.arep.mortgage       Official mortgage extension
  gov.hud.affordability   HUD affordability programs
  com.provider.custom     Provider-specific extension
```

### Extension Registration

```json
{
  "extensions": {
    "org.arep.screening": {
      "version": "2026-01",
      "operations": ["initiate_screening", "get_screening_result"]
    }
  }
}
```

### Available Extensions

| Extension | Purpose |
|-----------|---------|
| `org.arep.screening` | Tenant/buyer background checks |
| `org.arep.mortgage` | Mortgage rates, pre-approval |
| `org.arep.senior` | Senior housing (care levels) |
| `org.arep.commercial` | Commercial real estate |
| `gov.hud.affordability` | HUD programs (Section 8, FHA) |

## Implementation Patterns

### Provider Implementation

```typescript
// Express.js example
import express from 'express';

const app = express();

// Discovery manifest
app.get('/.well-known/arep', (req, res) => {
  res.json({
    arep: {
      version: '2026-01',
      provider: { name: 'My Provider', type: 'marketplace' },
      capabilities: {
        discovery: { version: '2026-01' },
        pricing: { version: '2026-01' }
      },
      transports: {
        rest: { endpoint: 'https://api.myprovider.com/arep/v1' }
      }
    }
  });
});

// Property search
app.get('/arep/v1/properties', async (req, res) => {
  const { transaction_type, city, state, price_max } = req.query;
  const properties = await searchProperties({ transaction_type, city, state, price_max });
  res.json({ total_results: properties.length, properties });
});
```

### Client Implementation

```typescript
import { AREPClient } from '@arep/sdk';

const client = new AREPClient({
  baseUrl: 'https://api.provider.com',
  apiKey: 'your-api-key'
});

// Search properties
const results = await client.discovery.search({
  transaction_type: 'for_sale',
  location: { city: 'Austin', state: 'TX' },
  price_max: 500000
});

// Get estimate
const estimate = await client.pricing.estimate({
  property_id: 'arep:example-123'
});
```

## Next Steps

- Review the [full specification](AREP_Specification.md)
- Check the [OpenAPI spec](../openapi/arep-v1.yaml) for REST details
- Explore [MCP tools](../mcp/tools.json) for AI integration
- See [examples/](../examples/) for reference implementations
