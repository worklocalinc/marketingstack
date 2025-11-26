# Repositories & Domains

Central reference for all WorkLocal Marketing Stack services, repositories, domains, and environments.

## Service Repositories

| Service | Repository | Description |
|---------|------------|-------------|
| **Marketing Stack** | `worklocalinc/marketingstack` | Architecture docs, contracts, central reference |
| **Affiliate Network** | `worklocalinc/AffiliateTracker` | Click tracking, conversions, revenue attribution |
| **Domain Steward** | `worklocalinc/domains.worklocal.ai` | Domain inventory, DNS, infrastructure management |
| **Messaging Core** | `work-local-inc/messigingcore` | Contacts, email/push sending, outbound messages |
| **Creative Generator** | `work-local-inc/creatives` | Templates, variants, AI creative generation |
| **Offer Creator** | `work-local-inc/superaffiliatesystem` | Offer blueprints, monetization flows, catalog |
| **Email Seeder** | `work-local-inc/seed` | Test emails, QA workflows, inbox monitoring |

## Domain Inventory

### Production Domains

| Domain | Service | Purpose |
|--------|---------|---------|
| `adjump.com` | Affiliate Network | Click tracking endpoints |
| `domains.adjump.com` | Domain Steward | Domain management API |
| `messages.adjump.com` | Messaging Core | Contact & messaging API |
| `creativebuilder.adjump.com` | Creative Generator | Template & variant API |
| `houseoffers.adjump.com` | Offer Creator | Offer blueprint API |
| `seed.adjump.com` | Email Seeder | Test email management API |

### Tracking Domains (Managed by Domain Steward)

| Domain Pattern | Purpose | Example |
|----------------|---------|---------|
| `adjump.com/*` | Click tracking | `adjump.com/click` |
| `go.*` | Short redirect links | Configured per campaign |

### Email Sending Domains

| Domain | Purpose | Provider |
|--------|---------|----------|
| `messages.adjump.com` | Transactional email | Resend |

### Lander Domains

| Domain | Vertical | Notes |
|--------|----------|-------|
| Configured per campaign | Job, finance, health | Managed by Domain Steward |

## Environments

### Development

| Service | URL | Database |
|---------|-----|----------|
| Affiliate Network | `localhost:3001` | Local Postgres / Neon dev |
| Domain Steward | `localhost:3002` | Local Postgres / Neon dev |
| Messaging Core | `localhost:3003` | Local Postgres / Neon dev |
| Creative Generator | `localhost:3004` | Local Postgres / Neon dev |
| Offer Creator | `localhost:3005` | Local Postgres / Neon dev |
| Email Seeder | `localhost:3006` | Local Postgres / Neon dev |

### Staging

| Service | URL | Database |
|---------|-----|----------|
| Affiliate Network | `staging.adjump.com` | Neon staging branch |
| Domain Steward | `staging.domains.adjump.com` | Neon staging branch |
| Messaging Core | `staging.messages.adjump.com` | Neon staging branch |
| Creative Generator | `staging.creativebuilder.adjump.com` | Neon staging branch |
| Offer Creator | `staging.houseoffers.adjump.com` | Neon staging branch |
| Email Seeder | `staging.seed.adjump.com` | Neon staging branch |

### Production

| Service | URL | Database |
|---------|-----|----------|
| Affiliate Network | `adjump.com` | Neon production |
| Domain Steward | `domains.adjump.com` | Neon production |
| Messaging Core | `messages.adjump.com` | Neon production |
| Creative Generator | `creativebuilder.adjump.com` | Neon production |
| Offer Creator | `houseoffers.adjump.com` | Neon production |
| Email Seeder | `seed.adjump.com` | Neon production |

## Environment Variables by Service

### Common Variables (All Services)

```bash
NODE_ENV=development|staging|production
DATABASE_URL=postgres://...
API_KEY=...
LOG_LEVEL=debug|info|warn|error
```

### Affiliate Network

```bash
# Core
CLICK_DOMAIN=adjump.com
POSTBACK_SECRET=...

# Service URLs
MESSAGING_CORE_URL=https://messages.adjump.com
DOMAIN_STEWARD_URL=https://domains.adjump.com
```

### Domain Steward

```bash
# DNS Providers
CLOUDFLARE_API_TOKEN=...
CLOUDFLARE_ZONE_ID=...
NAMESILO_API_KEY=...

# Integrations
NEON_API_KEY=...
ADJUMP_API_KEY=...
GITHUB_TOKEN=...  # For DNS-as-code
```

### Messaging Core

```bash
# ESP (Email)
RESEND_API_KEY=...
FROM_EMAIL=hello@messages.adjump.com

# Web Push (VAPID)
VAPID_PUBLIC_KEY=...
VAPID_PRIVATE_KEY=...
VAPID_SUBJECT=mailto:push@adjump.com

# Service URLs
AFFILIATE_NETWORK_URL=https://adjump.com
DOMAIN_STEWARD_URL=https://domains.adjump.com
CREATIVE_GENERATOR_URL=https://creativebuilder.adjump.com
```

### Creative Generator

```bash
# AI Providers
OPENAI_API_KEY=...
ANTHROPIC_API_KEY=...

# Asset Storage
CDN_URL=https://cdn.adjump.com
ASSET_BUCKET=...

# Service URLs
MESSAGING_CORE_URL=https://messages.adjump.com
```

### Offer Creator

```bash
# Service URLs
AFFILIATE_NETWORK_URL=https://adjump.com
CREATIVE_GENERATOR_URL=https://creativebuilder.adjump.com
DOMAIN_STEWARD_URL=https://domains.adjump.com
MESSAGING_CORE_URL=https://messages.adjump.com
```

### Email Seeder

```bash
# Email Providers
GMAIL_CLIENT_ID=...
GMAIL_CLIENT_SECRET=...
IMAP_DEFAULT_HOST=...
IMAP_DEFAULT_PORT=993

# Security
ENCRYPTION_KEY=...  # For credential storage

# Service URLs
MESSAGING_CORE_URL=https://messages.adjump.com
AFFILIATE_NETWORK_URL=https://adjump.com
OFFER_CREATOR_URL=https://houseoffers.adjump.com
```

## Service Communication Matrix

| From → To | Endpoint | Purpose |
|-----------|----------|---------|
| Offer Creator → Affiliate Network | `POST /api/offers` | Sync offer blueprints |
| Messaging Core → Creative Generator | `POST /api/templates/query` | Get template for sending |
| Messaging Core → Affiliate Network | `POST /click` | Generate tracking URLs |
| Affiliate Network → Messaging Core | `POST /api/events/affiliate` | Send conversion events |
| All Services → Domain Steward | `GET /api/tracking-domains` | Get available domains |
| Email Seeder → Messaging Core | `POST /api/contacts` | Register test contacts |
| Email Seeder → Affiliate Network | `GET /api/conversions` | Verify test conversions |

## Claude Code Sessions

Each service has a dedicated Claude Code session with environment-specific configuration:

| Service | Session Branch Pattern | Notes |
|---------|------------------------|-------|
| Marketing Stack | `claude/update-marketing-stack-*` | Architecture & docs |
| Affiliate Network | `claude/affiliate-network-*` | Tracking implementation |
| Domain Steward | `claude/domain-steward-*` | DNS & domain management |
| Messaging Core | `claude/messaging-core-*` | Contact & messaging |
| Creative Generator | `claude/creative-generator-*` | Templates & AI |
| Offer Creator | `claude/offer-creator-*` | Blueprints & flows |
| Email Seeder | `claude/email-seeder-*` | Test email management |

## Quick Links

- Architecture Overview: `docs/architecture.md`
- ID Contracts: `docs/contracts-ids-and-urls.md`
- Event Contracts: `docs/contracts-events.md`
- AI Guidance: `ai/README.md`

## Adding New Services

When adding a new service:

1. Create repository under `github.com/worklocalinc/`
2. Add entry to this document
3. Update `README.md` service list
4. Update `docs/architecture.md` with boundaries
5. Add service-specific documentation in `services/` and `docs/`
6. Update `ai/README.md` with guidance
