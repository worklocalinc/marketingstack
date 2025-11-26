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
| `trk.worklocal.dev` | Affiliate Network | Click tracking endpoints |
| `domains.worklocal.ai` | Domain Steward | Domain management API |
| `msg.worklocal.dev` | Messaging Core | Contact & messaging API |
| `creative.worklocal.dev` | Creative Generator | Template & variant API |
| `offers.worklocal.dev` | Offer Creator | Offer blueprint API |
| `seeder.worklocal.dev` | Email Seeder | Test email management API |

### Tracking Domains (Managed by Domain Steward)

| Domain Pattern | Purpose | Example |
|----------------|---------|---------|
| `trk.*` | Click tracking | `trk.offers.example` |
| `go.*` | Short redirect links | `go.worklocal.dev/abc` |
| `t.*` | Tracking pixels | `t.worklocal.dev/px` |

### Email Sending Domains

| Domain | Purpose | Provider |
|--------|---------|----------|
| `mail.worklocal.dev` | Transactional email | Resend |
| `news.worklocal.dev` | Newsletter sending | Resend |
| `notify.worklocal.dev` | Notification emails | Resend |

### Lander Domains

| Domain | Vertical | Notes |
|--------|----------|-------|
| `*.offers.example` | General offers | Managed by Domain Steward |
| Vertical-specific | Job, finance, health | Configured per campaign |

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
| Affiliate Network | `trk.staging.worklocal.dev` | Neon staging branch |
| Domain Steward | `domains.staging.worklocal.ai` | Neon staging branch |
| Messaging Core | `msg.staging.worklocal.dev` | Neon staging branch |
| Creative Generator | `creative.staging.worklocal.dev` | Neon staging branch |
| Offer Creator | `offers.staging.worklocal.dev` | Neon staging branch |
| Email Seeder | `seeder.staging.worklocal.dev` | Neon staging branch |

### Production

| Service | URL | Database |
|---------|-----|----------|
| Affiliate Network | `trk.worklocal.dev` | Neon production |
| Domain Steward | `domains.worklocal.ai` | Neon production |
| Messaging Core | `msg.worklocal.dev` | Neon production |
| Creative Generator | `creative.worklocal.dev` | Neon production |
| Offer Creator | `offers.worklocal.dev` | Neon production |
| Email Seeder | `seeder.worklocal.dev` | Neon production |

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
CLICK_DOMAIN=trk.worklocal.dev
POSTBACK_SECRET=...

# Service URLs
MESSAGING_CORE_URL=https://msg.worklocal.dev
DOMAIN_STEWARD_URL=https://domains.worklocal.ai
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
FROM_EMAIL=hello@mail.worklocal.dev

# Web Push (VAPID)
VAPID_PUBLIC_KEY=...
VAPID_PRIVATE_KEY=...
VAPID_SUBJECT=mailto:push@worklocal.dev

# Service URLs
AFFILIATE_NETWORK_URL=https://trk.worklocal.dev
DOMAIN_STEWARD_URL=https://domains.worklocal.ai
CREATIVE_GENERATOR_URL=https://creative.worklocal.dev
```

### Creative Generator

```bash
# AI Providers
OPENAI_API_KEY=...
ANTHROPIC_API_KEY=...

# Asset Storage
CDN_URL=https://cdn.worklocal.dev
ASSET_BUCKET=...

# Service URLs
MESSAGING_CORE_URL=https://msg.worklocal.dev
```

### Offer Creator

```bash
# Service URLs
AFFILIATE_NETWORK_URL=https://trk.worklocal.dev
CREATIVE_GENERATOR_URL=https://creative.worklocal.dev
DOMAIN_STEWARD_URL=https://domains.worklocal.ai
MESSAGING_CORE_URL=https://msg.worklocal.dev
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
MESSAGING_CORE_URL=https://msg.worklocal.dev
AFFILIATE_NETWORK_URL=https://trk.worklocal.dev
OFFER_CREATOR_URL=https://offers.worklocal.dev
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
