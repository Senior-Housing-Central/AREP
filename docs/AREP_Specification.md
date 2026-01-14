# AI Real Estate Protocol (AREP)

**Version:** 1.0.0-draft  
**Status:** Draft Specification  
**Date:** January 2026

---

## Abstract

The AI Real Estate Protocol (AREP) is an open standard enabling AI agents to discover properties, get valuations, submit applications or offers, and complete real estate transactions. AREP supports both **rental** and **sale** transactions through a unified protocol. Modeled after Google's Universal Commerce Protocol (UCP), AREP provides a transport-agnostic interface compatible with REST APIs, Model Context Protocol (MCP), and Agent-to-Agent (A2A) communication.

---

## 1. Introduction

### 1.1 Problem Statement

Real estate remains one of the least digitally integrated industries:

- **N×N Integration Problem**: Every platform (Zillow, Redfin, Realtor.com, MLS systems, property managers) has proprietary APIs
- **No Universal Transaction Standard**: Rental applications and purchase offers use incompatible formats
- **AI Agent Barriers**: AI assistants cannot programmatically search, compare, or transact across platforms
- **Fragmented Data**: Property data locked in silos (MLS, Yardi, RealPage, Zillow) with no interoperability

### 1.2 Solution

AREP defines a universal protocol for:

| Transaction Type | Capabilities |
|------------------|--------------|
| **Rentals** | Search, pricing, applications, lease signing, payments |
| **Sales** | Search, valuations, offers, contracts, closing |

### 1.3 Design Principles

| Principle | Description |
|-----------|-------------|
| **Unified Model** | Single protocol for rentals and sales |
| **Transport Agnostic** | Works over REST, MCP, A2A, GraphQL |
| **Incrementally Adoptable** | Providers implement only needed capabilities |
| **Extension-Friendly** | Vertical-specific needs via extension namespaces |
| **Standards-Aligned** | Builds on RESO Data Dictionary 2.0 |

---

## 2. Architecture

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

### 2.1 Core Capabilities

| Capability | Rentals | Sales | Description |
|------------|---------|-------|-------------|
| **Discovery** | ✓ | ✓ | Property search, details, availability |
| **Pricing** | ✓ | ✓ | Rent estimates, valuations, comps, market data |
| **Offer** | Application | Purchase Offer | Submit rental application or purchase offer |
| **Transaction** | Lease | Contract/Closing | Complete the transaction |

---

## 3. Discovery Protocol

### 3.1 Well-Known Endpoint

```
GET /.well-known/arep
```

### 3.2 Manifest Schema

```json
{
  "arep": {
    "version": "2026-01",
    "provider": {
      "name": "Example Realty",
      "type": "marketplace",
      "transaction_types": ["rental", "sale"],
      "regions": ["US"]
    },
    "capabilities": {
      "discovery": { "version": "2026-01" },
      "pricing": { "version": "2026-01" },
      "offer": { 
        "version": "2026-01",
        "supported_types": ["rental_application", "purchase_offer"]
      },
      "transaction": { "version": "2026-01" }
    },
    "extensions": {
      "org.arep.screening": { "version": "2026-01" },
      "org.arep.mortgage": { "version": "2026-01" }
    },
    "transports": {
      "rest": {
        "endpoint": "https://api.example.com/arep/v1"
      },
      "mcp": {
        "endpoint": "https://api.example.com/arep/mcp"
      },
      "a2a": {
        "agent_card": "https://api.example.com/.well-known/agent.json"
      }
    }
  }
}
```

---

## 4. Core Schemas

### 4.1 Property Schema

Unified schema supporting both rentals and sales.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://arep.dev/schemas/property.json",
  "title": "AREP Property",
  "type": "object",
  "required": ["property_id", "transaction_type", "property_type", "status", "address"],
  "properties": {
    "property_id": {
      "type": "string",
      "pattern": "^arep:[a-z0-9-]+$",
      "description": "Universal Property ID"
    },
    "transaction_type": {
      "type": "string",
      "enum": ["for_rent", "for_sale"],
      "description": "Rental or sale listing"
    },
    "property_type": {
      "type": "string",
      "enum": [
        "single_family",
        "condo",
        "townhouse",
        "multi_family",
        "apartment",
        "manufactured",
        "land",
        "commercial"
      ]
    },
    "status": {
      "type": "string",
      "enum": [
        "active",
        "pending",
        "contingent",
        "sold",
        "leased",
        "off_market",
        "coming_soon"
      ]
    },
    "address": {
      "$ref": "#/$defs/address"
    },
    "pricing": {
      "type": "object",
      "properties": {
        "list_price": {
          "type": "number",
          "description": "Sale price or monthly rent"
        },
        "price_per_sqft": {
          "type": "number"
        },
        "currency": {
          "type": "string",
          "default": "USD"
        },
        "deposit": {
          "type": "number",
          "description": "Security deposit (rentals)"
        },
        "hoa_fee": {
          "type": "number",
          "description": "Monthly HOA fee"
        },
        "tax_annual": {
          "type": "number",
          "description": "Annual property tax"
        }
      }
    },
    "details": {
      "type": "object",
      "properties": {
        "bedrooms": { "type": "integer" },
        "bathrooms": { "type": "number" },
        "sqft": { "type": "integer" },
        "lot_sqft": { "type": "integer" },
        "year_built": { "type": "integer" },
        "stories": { "type": "integer" },
        "garage_spaces": { "type": "integer" },
        "parking_type": {
          "type": "string",
          "enum": ["none", "street", "driveway", "carport", "garage", "covered"]
        }
      }
    },
    "features": {
      "type": "array",
      "items": { "type": "string" },
      "description": "Property features and amenities"
    },
    "media": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "type": {
            "type": "string",
            "enum": ["photo", "video", "virtual_tour", "floor_plan", "3d_model"]
          },
          "url": { "type": "string", "format": "uri" },
          "caption": { "type": "string" }
        }
      }
    },
    "availability": {
      "type": "object",
      "properties": {
        "available_date": { "type": "string", "format": "date" },
        "lease_terms": {
          "type": "array",
          "items": { "type": "integer" },
          "description": "Available lease terms in months"
        },
        "pets_allowed": { "type": "boolean" },
        "pet_deposit": { "type": "number" }
      }
    },
    "sale_details": {
      "type": "object",
      "description": "Sale-specific details",
      "properties": {
        "days_on_market": { "type": "integer" },
        "original_list_price": { "type": "number" },
        "price_change_date": { "type": "string", "format": "date" },
        "open_house": {
          "type": "array",
          "items": {
            "type": "object",
            "properties": {
              "date": { "type": "string", "format": "date" },
              "start_time": { "type": "string" },
              "end_time": { "type": "string" }
            }
          }
        }
      }
    },
    "listing_agent": {
      "type": "object",
      "properties": {
        "name": { "type": "string" },
        "phone": { "type": "string" },
        "email": { "type": "string", "format": "email" },
        "brokerage": { "type": "string" },
        "license_number": { "type": "string" }
      }
    },
    "mls": {
      "type": "object",
      "description": "MLS metadata",
      "properties": {
        "mls_id": { "type": "string" },
        "mls_name": { "type": "string" },
        "listing_id": { "type": "string" }
      }
    },
    "timestamps": {
      "type": "object",
      "properties": {
        "listed_at": { "type": "string", "format": "date-time" },
        "updated_at": { "type": "string", "format": "date-time" },
        "sold_at": { "type": "string", "format": "date-time" }
      }
    }
  },
  "$defs": {
    "address": {
      "type": "object",
      "required": ["street", "city", "state", "postal_code"],
      "properties": {
        "street": { "type": "string" },
        "unit": { "type": "string" },
        "city": { "type": "string" },
        "state": { "type": "string" },
        "postal_code": { "type": "string" },
        "county": { "type": "string" },
        "country": { "type": "string", "default": "US" },
        "coordinates": {
          "type": "object",
          "properties": {
            "lat": { "type": "number" },
            "lng": { "type": "number" }
          }
        }
      }
    }
  }
}
```

---

## 5. Capabilities

### 5.1 Discovery Capability

Search and retrieve property listings.

#### Operations

| Operation | Description |
|-----------|-------------|
| `search` | Search properties by criteria |
| `get` | Get property details by ID |
| `similar` | Find similar properties |
| `history` | Get property history (price changes, sales) |

#### search_properties

**Request:**
```json
{
  "transaction_type": "for_sale",
  "location": {
    "city": "Austin",
    "state": "TX"
  },
  "price_min": 300000,
  "price_max": 600000,
  "bedrooms_min": 3,
  "bathrooms_min": 2,
  "property_types": ["single_family", "townhouse"],
  "sqft_min": 1500,
  "year_built_min": 2000,
  "features": ["pool", "garage"],
  "sort_by": "price_asc",
  "page": 1,
  "page_size": 20
}
```

**Response:**
```json
{
  "total_results": 234,
  "properties": [
    {
      "property_id": "arep:zillow-123456",
      "transaction_type": "for_sale",
      "property_type": "single_family",
      "status": "active",
      "address": {
        "street": "456 Oak Lane",
        "city": "Austin",
        "state": "TX",
        "postal_code": "78701"
      },
      "pricing": {
        "list_price": 549000,
        "price_per_sqft": 275,
        "hoa_fee": 150,
        "tax_annual": 8500
      },
      "details": {
        "bedrooms": 4,
        "bathrooms": 2.5,
        "sqft": 2100,
        "lot_sqft": 8500,
        "year_built": 2015,
        "garage_spaces": 2
      },
      "sale_details": {
        "days_on_market": 12,
        "open_house": [
          {
            "date": "2026-01-18",
            "start_time": "13:00",
            "end_time": "16:00"
          }
        ]
      }
    }
  ]
}
```

---

### 5.2 Pricing Capability

Valuations, estimates, comparables, and market data.

#### Operations

| Operation | Description |
|-----------|-------------|
| `estimate` | Get property value estimate (Zestimate-style) |
| `rent_estimate` | Get rental price estimate |
| `comparables` | Get comparable sales/rentals |
| `market_trends` | Get market statistics for an area |
| `affordability` | Calculate affordability based on income |

#### estimate (Value Estimate)

**Request:**
```json
{
  "property_id": "arep:zillow-123456"
}
```

**Response:**
```json
{
  "property_id": "arep:zillow-123456",
  "estimate": {
    "value": 545000,
    "value_low": 518000,
    "value_high": 572000,
    "confidence": "high",
    "as_of": "2026-01-13"
  },
  "rent_estimate": {
    "monthly": 2800,
    "monthly_low": 2600,
    "monthly_high": 3100
  },
  "value_history": [
    { "date": "2025-01-01", "value": 520000 },
    { "date": "2026-01-01", "value": 545000 }
  ],
  "forecast": {
    "one_year": {
      "change_percent": 3.2,
      "value": 562400
    }
  }
}
```

#### comparables

**Request:**
```json
{
  "property_id": "arep:zillow-123456",
  "transaction_type": "sold",
  "radius_miles": 1,
  "limit": 10,
  "sold_within_days": 180
}
```

**Response:**
```json
{
  "subject_property": {
    "property_id": "arep:zillow-123456",
    "list_price": 549000,
    "sqft": 2100,
    "price_per_sqft": 261
  },
  "comparables": [
    {
      "property_id": "arep:mls-789012",
      "address": {
        "street": "123 Elm Street",
        "city": "Austin",
        "state": "TX"
      },
      "sold_price": 535000,
      "sold_date": "2025-11-15",
      "sqft": 2050,
      "price_per_sqft": 261,
      "bedrooms": 4,
      "bathrooms": 2,
      "distance_miles": 0.3,
      "similarity_score": 0.92
    }
  ],
  "market_summary": {
    "median_sold_price": 542000,
    "median_price_per_sqft": 265,
    "avg_days_on_market": 18
  }
}
```

#### market_trends

**Request:**
```json
{
  "location": {
    "city": "Austin",
    "state": "TX",
    "postal_code": "78701"
  },
  "transaction_type": "for_sale",
  "period": "12_months"
}
```

**Response:**
```json
{
  "location": {
    "city": "Austin",
    "state": "TX",
    "postal_code": "78701"
  },
  "period": "12_months",
  "sale_market": {
    "median_list_price": 525000,
    "median_sold_price": 515000,
    "median_price_per_sqft": 268,
    "inventory": 1250,
    "new_listings": 3400,
    "sold_count": 2890,
    "median_days_on_market": 21,
    "sale_to_list_ratio": 0.98,
    "price_change_yoy": 4.2
  },
  "rental_market": {
    "median_rent": 2400,
    "rent_change_yoy": 3.1,
    "vacancy_rate": 5.2
  },
  "trends": [
    { "month": "2025-01", "median_price": 505000 },
    { "month": "2025-06", "median_price": 510000 },
    { "month": "2026-01", "median_price": 525000 }
  ]
}
```

#### affordability

**Request:**
```json
{
  "annual_income": 150000,
  "monthly_debts": 500,
  "down_payment": 100000,
  "location": {
    "state": "TX"
  },
  "interest_rate": 6.5,
  "loan_term_years": 30
}
```

**Response:**
```json
{
  "max_purchase_price": 625000,
  "comfortable_price": 550000,
  "monthly_payment": {
    "principal_interest": 3312,
    "property_tax": 850,
    "insurance": 200,
    "hoa": 0,
    "total": 4362
  },
  "debt_to_income": 0.36,
  "down_payment_percent": 16,
  "loan_amount": 525000
}
```

---

### 5.3 Offer Capability

Submit rental applications or purchase offers.

#### Operations

| Operation | Type | Description |
|-----------|------|-------------|
| `create_application` | Rental | Start rental application |
| `submit_application` | Rental | Submit completed application |
| `create_offer` | Sale | Create purchase offer |
| `submit_offer` | Sale | Submit offer to seller |
| `get_status` | Both | Check application/offer status |
| `counter` | Sale | Respond to counter-offer |

#### Rental Application Schema

```json
{
  "type": "rental_application",
  "application_id": "arep:app:rental-abc123",
  "property_id": "arep:apt-456",
  "status": "draft",
  "applicant": {
    "name": {
      "first": "John",
      "last": "Smith"
    },
    "email": "john@example.com",
    "phone": "+1-555-123-4567",
    "date_of_birth": "1985-06-15",
    "ssn_last_four": "1234",
    "current_address": {
      "street": "789 Current St",
      "city": "Austin",
      "state": "TX",
      "postal_code": "78702"
    }
  },
  "employment": {
    "status": "employed",
    "employer": "Tech Corp",
    "position": "Engineer",
    "monthly_income": 8500,
    "start_date": "2020-03-01"
  },
  "rental_history": [
    {
      "address": { "street": "...", "city": "...", "state": "..." },
      "landlord_name": "Jane Doe",
      "landlord_phone": "+1-555-987-6543",
      "monthly_rent": 1800,
      "dates": { "from": "2022-01", "to": "present" }
    }
  ],
  "desired_move_in": "2026-03-01",
  "desired_lease_term": 12,
  "documents": [],
  "consents": {
    "background_check": false,
    "credit_check": false
  }
}
```

#### Purchase Offer Schema

```json
{
  "type": "purchase_offer",
  "offer_id": "arep:offer:purchase-xyz789",
  "property_id": "arep:zillow-123456",
  "status": "draft",
  "buyer": {
    "name": "John Smith",
    "email": "john@example.com",
    "phone": "+1-555-123-4567",
    "agent": {
      "name": "Sarah Johnson",
      "brokerage": "Realty Partners",
      "license": "TX-12345",
      "email": "sarah@realtypartners.com"
    }
  },
  "offer_price": 540000,
  "earnest_money": 15000,
  "down_payment_percent": 20,
  "financing": {
    "type": "conventional",
    "pre_approved": true,
    "lender": "First National Bank",
    "pre_approval_amount": 600000,
    "pre_approval_expiration": "2026-03-15"
  },
  "contingencies": {
    "inspection": {
      "included": true,
      "days": 10
    },
    "appraisal": {
      "included": true
    },
    "financing": {
      "included": true,
      "days": 21
    },
    "sale_of_home": {
      "included": false
    }
  },
  "closing_date": "2026-03-15",
  "possession_date": "2026-03-15",
  "inclusions": ["refrigerator", "washer", "dryer"],
  "exclusions": ["dining room chandelier"],
  "additional_terms": "Seller to provide home warranty",
  "expiration": "2026-01-15T18:00:00Z"
}
```

#### Offer Status Flow (Sales)

```
┌────────┐   submit    ┌───────────┐
│ draft  │────────────▶│ submitted │
└────────┘             └─────┬─────┘
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
         ▼                   ▼                   ▼
   ┌──────────┐       ┌───────────┐       ┌──────────┐
   │ accepted │       │ countered │       │ rejected │
   └────┬─────┘       └─────┬─────┘       └──────────┘
        │                   │
        │                   ▼
        │            ┌────────────┐
        │            │ negotiating│◀──┐
        │            └─────┬──────┘   │
        │                  │          │
        │         ┌────────┴────────┐ │
        │         ▼                 ▼ │
        │   ┌──────────┐     ┌─────────┐
        │   │ accepted │     │counter  │──┘
        │   └────┬─────┘     └─────────┘
        │        │
        ▼        ▼
   ┌─────────────────┐
   │   under_contract │
   └────────┬────────┘
            │
            ▼
   ┌─────────────────┐
   │     closed      │
   └─────────────────┘
```

---

### 5.4 Transaction Capability

Complete leases or purchase closings.

#### Operations

| Operation | Type | Description |
|-----------|------|-------------|
| `create_lease` | Rental | Generate lease from approved application |
| `sign_lease` | Rental | E-sign lease document |
| `create_contract` | Sale | Generate purchase contract |
| `sign_contract` | Sale | E-sign purchase contract |
| `get_documents` | Both | Get transaction documents |
| `schedule_closing` | Sale | Schedule closing appointment |

---

## 6. MCP Tool Definitions

### 6.1 search_properties

```json
{
  "name": "search_properties",
  "description": "Search real estate listings for sale or rent",
  "inputSchema": {
    "type": "object",
    "required": ["transaction_type", "location"],
    "properties": {
      "transaction_type": {
        "type": "string",
        "enum": ["for_sale", "for_rent"],
        "description": "Search for properties to buy or rent"
      },
      "location": {
        "type": "object",
        "description": "Search location",
        "properties": {
          "city": { "type": "string" },
          "state": { "type": "string" },
          "postal_code": { "type": "string" },
          "coordinates": {
            "type": "object",
            "properties": {
              "lat": { "type": "number" },
              "lng": { "type": "number" },
              "radius_miles": { "type": "number" }
            }
          }
        }
      },
      "price_min": { "type": "number" },
      "price_max": { "type": "number" },
      "bedrooms_min": { "type": "integer" },
      "bathrooms_min": { "type": "number" },
      "sqft_min": { "type": "integer" },
      "sqft_max": { "type": "integer" },
      "year_built_min": { "type": "integer" },
      "property_types": {
        "type": "array",
        "items": {
          "type": "string",
          "enum": ["single_family", "condo", "townhouse", "multi_family", "apartment", "land"]
        }
      },
      "features": {
        "type": "array",
        "items": { "type": "string" }
      },
      "status": {
        "type": "array",
        "items": {
          "type": "string",
          "enum": ["active", "pending", "contingent", "coming_soon"]
        }
      },
      "sort_by": {
        "type": "string",
        "enum": ["price_asc", "price_desc", "newest", "sqft", "days_on_market"]
      },
      "page": { "type": "integer", "default": 1 },
      "page_size": { "type": "integer", "default": 20 }
    }
  }
}
```

### 6.2 get_property_details

```json
{
  "name": "get_property_details",
  "description": "Get detailed information about a specific property",
  "inputSchema": {
    "type": "object",
    "required": ["property_id"],
    "properties": {
      "property_id": {
        "type": "string",
        "description": "AREP property ID"
      },
      "include": {
        "type": "array",
        "items": {
          "type": "string",
          "enum": ["estimate", "history", "comparables", "schools", "walkability"]
        },
        "description": "Additional data to include"
      }
    }
  }
}
```

### 6.3 get_estimate

```json
{
  "name": "get_estimate",
  "description": "Get property value or rent estimate",
  "inputSchema": {
    "type": "object",
    "required": ["property_id"],
    "properties": {
      "property_id": {
        "type": "string",
        "description": "AREP property ID or address"
      },
      "estimate_type": {
        "type": "string",
        "enum": ["sale", "rent", "both"],
        "default": "both"
      }
    }
  }
}
```

### 6.4 get_comparables

```json
{
  "name": "get_comparables",
  "description": "Get comparable properties (comps) for valuation",
  "inputSchema": {
    "type": "object",
    "required": ["property_id"],
    "properties": {
      "property_id": { "type": "string" },
      "transaction_type": {
        "type": "string",
        "enum": ["sold", "for_sale", "rented", "for_rent"]
      },
      "radius_miles": { "type": "number", "default": 1 },
      "limit": { "type": "integer", "default": 10 },
      "days": { "type": "integer", "default": 180 }
    }
  }
}
```

### 6.5 get_market_trends

```json
{
  "name": "get_market_trends",
  "description": "Get real estate market statistics for an area",
  "inputSchema": {
    "type": "object",
    "required": ["location"],
    "properties": {
      "location": {
        "type": "object",
        "properties": {
          "city": { "type": "string" },
          "state": { "type": "string" },
          "postal_code": { "type": "string" },
          "county": { "type": "string" }
        }
      },
      "period": {
        "type": "string",
        "enum": ["1_month", "3_months", "6_months", "12_months", "5_years"],
        "default": "12_months"
      },
      "property_type": {
        "type": "string",
        "enum": ["all", "single_family", "condo", "townhouse"]
      }
    }
  }
}
```

### 6.6 calculate_affordability

```json
{
  "name": "calculate_affordability",
  "description": "Calculate how much home a buyer can afford",
  "inputSchema": {
    "type": "object",
    "required": ["annual_income"],
    "properties": {
      "annual_income": { "type": "number" },
      "monthly_debts": { "type": "number", "default": 0 },
      "down_payment": { "type": "number" },
      "down_payment_percent": { "type": "number" },
      "interest_rate": { "type": "number" },
      "loan_term_years": { "type": "integer", "default": 30 },
      "location": {
        "type": "object",
        "properties": {
          "state": { "type": "string" },
          "postal_code": { "type": "string" }
        }
      },
      "include_taxes": { "type": "boolean", "default": true },
      "include_insurance": { "type": "boolean", "default": true }
    }
  }
}
```

### 6.7 create_offer

```json
{
  "name": "create_offer",
  "description": "Create a rental application or purchase offer",
  "inputSchema": {
    "type": "object",
    "required": ["property_id", "offer_type"],
    "properties": {
      "property_id": { "type": "string" },
      "offer_type": {
        "type": "string",
        "enum": ["rental_application", "purchase_offer"]
      },
      "applicant": {
        "type": "object",
        "properties": {
          "name": { "type": "string" },
          "email": { "type": "string" },
          "phone": { "type": "string" }
        }
      },
      "offer_price": {
        "type": "number",
        "description": "Purchase offer price (sales only)"
      },
      "move_in_date": { "type": "string", "format": "date" },
      "closing_date": { "type": "string", "format": "date" }
    }
  }
}
```

### 6.8 get_offer_status

```json
{
  "name": "get_offer_status",
  "description": "Check status of rental application or purchase offer",
  "inputSchema": {
    "type": "object",
    "required": ["offer_id"],
    "properties": {
      "offer_id": { "type": "string" }
    }
  }
}
```

---

## 7. A2A Agent Card

```json
{
  "name": "AREP Real Estate Agent",
  "description": "AI agent for property search, valuations, and transactions across rental and sale markets",
  "url": "https://api.provider.com/a2a",
  "version": "1.0.0",
  "capabilities": {
    "streaming": true,
    "pushNotifications": true
  },
  "defaultInputModes": ["text", "application/json"],
  "defaultOutputModes": ["text", "application/json"],
  "skills": [
    {
      "id": "property_search",
      "name": "Property Search",
      "description": "Search homes for sale or rent by location, price, and features",
      "tags": ["real_estate", "search", "housing", "rental", "sale"],
      "examples": [
        "Find 3-bedroom homes for sale in Austin under $500k",
        "Show me apartments for rent near downtown Seattle",
        "What condos are available in Miami Beach?"
      ]
    },
    {
      "id": "property_valuation",
      "name": "Property Valuation",
      "description": "Get property value estimates, rent estimates, and comparable sales",
      "tags": ["real_estate", "valuation", "pricing", "comps"],
      "examples": [
        "What's the estimated value of 123 Main St?",
        "How much could I rent my house for?",
        "Show me comparable sales in this neighborhood"
      ]
    },
    {
      "id": "market_analysis",
      "name": "Market Analysis",
      "description": "Get real estate market trends, statistics, and forecasts",
      "tags": ["real_estate", "market", "trends", "analysis"],
      "examples": [
        "How is the Austin housing market doing?",
        "Are home prices going up in Denver?",
        "What's the average rent in San Francisco?"
      ]
    },
    {
      "id": "affordability",
      "name": "Affordability Calculator",
      "description": "Calculate home buying or rental affordability",
      "tags": ["real_estate", "mortgage", "affordability", "finance"],
      "examples": [
        "How much house can I afford with $120k income?",
        "What would my mortgage payment be on a $400k home?",
        "Can I afford a $2500/month apartment?"
      ]
    },
    {
      "id": "offer_submission",
      "name": "Offer Submission",
      "description": "Submit rental applications or purchase offers",
      "tags": ["real_estate", "application", "offer", "transaction"],
      "examples": [
        "Start an application for this apartment",
        "Submit an offer of $450k on this house",
        "Check the status of my offer"
      ]
    }
  ],
  "authentication": {
    "schemes": ["oauth2"]
  }
}
```

---

## 8. Extensions

### 8.1 Extension Namespaces

| Namespace | Purpose |
|-----------|---------|
| `org.arep.screening` | Tenant/buyer screening (credit, background) |
| `org.arep.mortgage` | Mortgage pre-approval, rates, calculators |
| `org.arep.title` | Title search, insurance |
| `org.arep.inspection` | Home inspection scheduling, reports |
| `org.arep.moving` | Moving services, utilities setup |
| `gov.hud.affordability` | HUD programs (Section 8, FHA) |
| `org.arep.senior` | Senior housing (care levels, accessibility) |
| `org.arep.commercial` | Commercial real estate |

### 8.2 Extension: org.arep.mortgage

```json
{
  "extension": "org.arep.mortgage",
  "version": "2026-01",
  "operations": {
    "get_rates": {
      "description": "Get current mortgage rates",
      "input": {
        "loan_type": ["conventional", "fha", "va", "jumbo"],
        "term_years": [15, 30],
        "location": { "state": "string" }
      },
      "output": {
        "rates": [{
          "loan_type": "string",
          "term_years": "integer",
          "rate": "number",
          "apr": "number",
          "points": "number"
        }]
      }
    },
    "calculate_payment": {
      "description": "Calculate monthly mortgage payment",
      "input": {
        "loan_amount": "number",
        "interest_rate": "number",
        "term_years": "integer",
        "property_tax_annual": "number",
        "insurance_annual": "number",
        "hoa_monthly": "number",
        "pmi_rate": "number"
      },
      "output": {
        "monthly_payment": {
          "principal_interest": "number",
          "property_tax": "number",
          "insurance": "number",
          "hoa": "number",
          "pmi": "number",
          "total": "number"
        },
        "amortization_schedule": "array"
      }
    },
    "check_preapproval": {
      "description": "Check pre-approval status or get pre-approved",
      "input": {
        "applicant": "object",
        "income": "object",
        "assets": "object",
        "debts": "object"
      }
    }
  }
}
```

### 8.3 Extension: org.arep.screening

```json
{
  "extension": "org.arep.screening",
  "version": "2026-01",
  "operations": {
    "initiate_screening": {
      "description": "Start background/credit check",
      "input": {
        "application_id": "string",
        "screening_types": ["credit", "criminal", "eviction", "employment", "income"],
        "consent_token": "string"
      }
    },
    "get_screening_result": {
      "description": "Get screening results",
      "output": {
        "recommendation": "approve | conditional | decline",
        "credit_score": "integer",
        "factors": "array"
      }
    }
  }
}
```

---

## 9. Transport Bindings

### 9.1 REST API

```
Base: https://api.provider.com/arep/v1

# Discovery
GET    /properties?transaction_type=for_sale&city=Austin&...
GET    /properties/{property_id}
GET    /properties/{property_id}/similar
GET    /properties/{property_id}/history

# Pricing
GET    /estimate/{property_id}
GET    /comparables/{property_id}
GET    /market/{location}
POST   /affordability

# Offers
POST   /applications              (rental)
POST   /offers                    (purchase)
GET    /applications/{id}
GET    /offers/{id}
PUT    /offers/{id}
POST   /offers/{id}/submit

# Transaction
POST   /leases                    (rental)
POST   /contracts                 (purchase)
GET    /documents/{transaction_id}
POST   /sign/{document_id}
```

### 9.2 MCP (JSON-RPC 2.0)

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "search_properties",
    "arguments": {
      "transaction_type": "for_sale",
      "location": { "city": "Austin", "state": "TX" },
      "price_max": 500000,
      "bedrooms_min": 3
    }
  },
  "id": 1
}
```

### 9.3 A2A (Task-based)

```json
{
  "jsonrpc": "2.0",
  "method": "message/send",
  "params": {
    "message": {
      "role": "user",
      "parts": [{
        "type": "text",
        "text": "Find 3-bedroom homes for sale in Austin under $500k"
      }]
    }
  },
  "id": "task-abc123"
}
```

---

## 10. Security

### 10.1 Authentication

| Method | Use Case |
|--------|----------|
| OAuth 2.0 Client Credentials | B2B provider access |
| OAuth 2.0 Authorization Code | End-user authorization |
| API Keys | Development/testing |

### 10.2 Scopes

```
property:read          Search and view properties
property:write         Create/update listings
estimate:read          Get valuations and comps
application:read       View applications
application:write      Create/submit applications
offer:read             View purchase offers
offer:write            Create/submit offers
transaction:read       View documents
transaction:sign       Sign documents
screening:read         View screening results
```

### 10.3 Data Privacy

- Minimal disclosure by default
- Consent tracking for all PII access
- Support for CCPA/GDPR deletion requests
- Sensitive data (SSN, financials) requires elevated consent

---

## 11. Error Codes

| Code | HTTP | Description |
|------|------|-------------|
| `INVALID_REQUEST` | 400 | Malformed request |
| `UNAUTHORIZED` | 401 | Invalid credentials |
| `FORBIDDEN` | 403 | Insufficient scope |
| `NOT_FOUND` | 404 | Resource not found |
| `CONFLICT` | 409 | State conflict (e.g., offer already accepted) |
| `VALIDATION_ERROR` | 422 | Schema validation failed |
| `RATE_LIMITED` | 429 | Too many requests |

---

## 12. RESO Alignment

AREP aligns with RESO Data Dictionary 2.0:

| AREP | RESO |
|------|------|
| `property_id` | `ListingKey` |
| `transaction_type` | `PropertySubType` |
| `status` | `StandardStatus` |
| `pricing.list_price` | `ListPrice` |
| `details.bedrooms` | `BedroomsTotal` |
| `details.bathrooms` | `BathroomsTotalInteger` |
| `details.sqft` | `LivingArea` |
| `details.year_built` | `YearBuilt` |
| `sale_details.days_on_market` | `DaysOnMarket` |

---

## 13. Roadmap

### Phase 1: Foundation (Q1-Q2 2026)
- [ ] JSON Schema publication
- [ ] REST API (OpenAPI 3.0)
- [ ] MCP tool definitions
- [ ] Python/TypeScript SDKs
- [ ] Reference implementation

### Phase 2: Integrations (Q3-Q4 2026)
- [ ] MLS connectivity
- [ ] Zillow/Redfin bridge adapters
- [ ] Mortgage extension
- [ ] Screening extension
- [ ] A2A Agent Cards

### Phase 3: Ecosystem (2027)
- [ ] Industry working group
- [ ] Certification program
- [ ] International expansion

---

## Appendix: Feature Comparison

| Feature | AREP | Zillow API | Redfin | MLS (RESO) |
|---------|------|------------|--------|------------|
| For Sale Listings | ✓ | ✓ | ✓ | ✓ |
| For Rent Listings | ✓ | ✓ | ✗ | Partial |
| Value Estimates | ✓ | ✓ | ✓ | ✗ |
| Rent Estimates | ✓ | ✓ | ✗ | ✗ |
| Comparables | ✓ | ✓ | ✓ | ✓ |
| Market Trends | ✓ | ✓ | ✓ | ✗ |
| Applications | ✓ | ✗ | ✗ | ✗ |
| Purchase Offers | ✓ | ✗ | ✗ | ✗ |
| MCP Support | ✓ | ✗ | ✗ | ✗ |
| A2A Support | ✓ | ✗ | ✗ | ✗ |
| Open Standard | ✓ | ✗ | ✗ | ✓ |

---

**License:** Apache 2.0  
**Repository:** https://github.com/arep-dev/spec  
**Contact:** spec@arep.dev
