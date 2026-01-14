# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [1.0.0-draft] - 2026-01-13

### Added

- Initial AREP specification (version 2026-01)
- Core capabilities:
  - **Discovery** - Property search, details, similar properties, history
  - **Pricing** - Value estimates, rent estimates, comparables, market trends, affordability
  - **Offer** - Rental applications and purchase offers
  - **Transaction** - Lease and contract management
- JSON Schemas:
  - Property schema with unified rental/sale support
  - Discovery manifest schema for `/.well-known/arep`
  - Rental application schema
  - Purchase offer schema
  - A2A Agent Card schema
- OpenAPI 3.0 REST API specification
- MCP tool definitions for AI assistant integration
- Transport bindings:
  - REST API
  - MCP (Model Context Protocol)
  - A2A (Agent-to-Agent)
- Extension namespaces:
  - `org.arep.screening` - Tenant/buyer screening
  - `org.arep.mortgage` - Mortgage integration
  - `org.arep.senior` - Senior housing
  - `org.arep.commercial` - Commercial real estate
  - `gov.hud.affordability` - HUD programs
- RESO Data Dictionary 2.0 alignment
- OAuth 2.0 authentication support
- Security scopes for fine-grained access control

### Security

- Minimal disclosure by default
- Consent tracking for PII access
- CCPA/GDPR deletion request support
- Elevated consent for sensitive data (SSN, financials)

[Unreleased]: https://github.com/arep-dev/spec/compare/v1.0.0-draft...HEAD
[1.0.0-draft]: https://github.com/arep-dev/spec/releases/tag/v1.0.0-draft
