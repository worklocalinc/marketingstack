# Domain Steward Service

This service owns domain inventory, DNS configuration, and how domains are used across the stack.

## Responsibilities

- Track all domains used by the system.
- Maintain DNS and infrastructure metadata (e.g., via Cloudflare).
- Define how each domain is used:
  - tracking, lander, email, CDN, etc.
- Integrate with registrars, Cloudflare, Neon, AdJump.

## Core Concepts

- **Domain** – e.g., `trk.example.com`, `offers.example.com`.
- **Domain Usage** – mapping between a domain and its role:
  - `role`: `tracking` | `lander` | `email` | `cdn` | ...
  - `service`: which service primarily uses it.

## Integration Points

- **Affiliate Network**:
  - Uses Domain Steward to determine:
    - `TRK_DOMAIN` values for tracking links.
    - Which brand/offer maps to which tracking domain.

- **Messaging Core**:
  - Uses configured tracking domains for link generation.
  - Uses email domains for ESP configuration.

## Database Schema (Conceptual)

```sql
-- Domains
domains:
  - id (domainId)
  - organizationId
  - domainName
  - registrar
  - registrarAccountId
  - status (active/pending/expired)
  - expirationDate
  - autoRenew
  - dnsProvider
  - metadata (JSON)
  - timestamps

-- Domain Usages
domain_usages:
  - id
  - domainId
  - role (tracking/lander/email/cdn)
  - service (affiliate-network/messaging-core)
  - priority
  - configuration (JSON)
  - isActive
  - timestamps

-- DNS Records
dns_records:
  - id
  - domainId
  - type (A/CNAME/MX/TXT/etc)
  - name
  - value
  - ttl
  - proxied (for Cloudflare)
  - timestamps

-- SSL Certificates
ssl_certificates:
  - id
  - domainId
  - provider
  - status
  - expirationDate
  - timestamps
```

## HTTP Endpoints (Planned)

### Domain Management

- `GET /api/domains`
  - List all domains with filters (status, role, expiration)

- `POST /api/domains`
  - Register or add a new domain

- `GET /api/domains/:id`
  - Get domain details including usage and DNS

- `PUT /api/domains/:id`
  - Update domain configuration

### Domain Usage

- `GET /api/domain-usages`
  - Query: `role`, `service`
  - Returns domains available for specific use cases

- `POST /api/domain-usages`
  - Assign a domain to a role/service

- `DELETE /api/domain-usages/:id`
  - Remove a domain usage assignment

### DNS Management

- `GET /api/domains/:id/dns`
  - Get DNS records for a domain

- `POST /api/domains/:id/dns`
  - Create DNS record

- `PUT /api/domains/:id/dns/:recordId`
  - Update DNS record

- `DELETE /api/domains/:id/dns/:recordId`
  - Delete DNS record

### Integration Endpoints

- `GET /api/tracking-domains`
  - Returns available tracking domains for the Affiliate Network

- `GET /api/email-domains`
  - Returns configured email sending domains for Messaging Core

## Environment Variables

```env
# Database
DATABASE_URL=postgresql://...

# Cloudflare Integration
CLOUDFLARE_ACCOUNT_ID=...
CLOUDFLARE_API_TOKEN=...

# Namesilo Integration
NAMESILO_API_KEY=...

# Neon Integration
NEON_API_KEY=...
NEON_DATABASE_URL=...

# GitHub Integration (for DNS as Code)
GITHUB_TOKEN=...
GITHUB_ACCOUNT=...

# AdJump Integration
ADJUMP_API_KEY=...

# Service Authentication
SERVICE_AUTH_TOKEN=...
AUTH_CORE_BASE_URL=...
AUTH_CORE_GLOBAL_API_KEY=...

# Encryption
CREDENTIALS_ENCRYPTION_KEY=...
```

## API Examples

### Get Tracking Domains

```http
GET /api/tracking-domains
Authorization: Bearer <TOKEN>
```

Response:
```json
{
  "domains": [
    {
      "domainId": "dom_123",
      "domainName": "trk.offers.example",
      "priority": 1,
      "configuration": {
        "ssl": true,
        "redirectMethod": "302"
      }
    },
    {
      "domainId": "dom_124",
      "domainName": "track.deals.example",
      "priority": 2,
      "configuration": {
        "ssl": true,
        "redirectMethod": "302"
      }
    }
  ]
}
```

### Create DNS Record

```http
POST /api/domains/dom_123/dns
Content-Type: application/json
Authorization: Bearer <TOKEN>

{
  "type": "A",
  "name": "@",
  "value": "192.168.1.1",
  "ttl": 3600,
  "proxied": true
}
```

### Assign Domain Usage

```http
POST /api/domain-usages
Content-Type: application/json
Authorization: Bearer <TOKEN>

{
  "domainId": "dom_125",
  "role": "lander",
  "service": "affiliate-network",
  "priority": 1,
  "configuration": {
    "vertical": "finance",
    "geoTargeting": ["US", "CA"]
  }
}
```

## Integration with External Services

### Cloudflare

- Manage DNS records
- SSL certificate provisioning
- DDoS protection and CDN

### Namesilo

- Domain registration
- Domain renewal
- WHOIS privacy

### Neon

- Database provisioning for multi-tenant setups
- Backup and restore operations

### AdJump

- Tracking domain configuration
- Offer URL management