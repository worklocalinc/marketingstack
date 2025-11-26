# WorkLocal Marketing Stack

This repo documents the core architecture for James's performance marketing platform.
It is designed so **humans and AI agents** can understand how the pieces fit together and extend them safely.

## Repositories & Domains

| Service | Repository | Production Domain | Purpose |
|---------|------------|-------------------|---------|
| Affiliate Network | `worklocalinc/AffiliateTracker` | `trk.worklocal.dev` | Click/conversion tracking, revenue attribution |
| Domain Steward | `worklocalinc/domains.worklocal.ai` | `domains.worklocal.ai` | Domain inventory, DNS management |
| Messaging Core | `work-local-inc/messigingcore` | `msg.worklocal.dev` | Contacts, email/push sending |
| Creative Generator | `work-local-inc/creatives` | `creative.worklocal.dev` | Templates, AI creative generation |
| Offer Creator | `work-local-inc/superaffiliatesystem` | `offers.worklocal.dev` | Offer blueprints, monetization flows |
| Email Seeder | `work-local-inc/seed` | `seeder.worklocal.dev` | Test emails, signup testing, QA |

> See `docs/repos-and-domains.md` for full domain inventory and environment details.

## High-Level Mental Model

Six main services:

1. **Affiliate Network Platform**  
   - Owns offers, affiliates, advertisers, clicks, conversions, revenue, payouts.  
   - Tracks traffic with click IDs and propagates subIDs for attribution.

2. **Domain Steward**  
   - Owns domains, DNS, infra, and how domains are used across the stack.  
   - Integrates with registrars, Cloudflare, Neon, AdJump, etc.

3. **Messaging Core**  
   - Owns contacts and consent.  
   - Sends messages across channels (email, web push, SMS later).  
   - Tracks outbound messages and ties them to clicks and conversions via tracking URLs.

4. **Creative Generator (Creative Studio)**  
   - Owns templates, variants, and creative assets for all channels.  
   - Provides AI-assisted creative generation and optimization.  
   - Does NOT send messages or track performance directly.

5. **Offer Creator (Offer Orchestrator)**
   - Designs offer blueprints and monetization flows.
   - Syncs offers into Affiliate Network and maintains catalog.
   - Connects offers to domains, creatives, and messaging sequences.

6. **Email Seeder**
   - Manages pre-configured test email accounts for QA workflows.
   - Provides ready-to-use emails for signups, offer testing, and automation validation.
   - Does NOT send production messages (that's Messaging Core).

Always keep this split:

- **Messaging Core** = who we talked to and what we sent.
- **Affiliate Network** = what they did and how much money it made.
- **Domain Steward** = where it lives and under which domains.
- **Creative Generator** = what to say and how it looks.
- **Offer Creator** = which offers exist and how they're orchestrated.
- **Email Seeder** = test accounts for QA and validation.

## Canonical IDs

These IDs must be preserved end-to-end:

- `contactId` → Messaging Core → `contacts.id`
- `messageId` → Messaging Core → `outbound_messages.id`
- `subscriptionId` → Messaging Core → `push_subscriptions.id`
- `clickId` → Affiliate Network → `clicks.clickId`
- `conversionId` → Affiliate Network → `conversions.id`
- `creativeTemplateId` → Creative Generator → `creative_templates.id`
- `creativeVariantId` → Creative Generator → `creative_variants.id`
- `offerBlueprintId` → Offer Creator → `offer_blueprints.id`
- `offerFlowId` → Offer Creator → `offer_flows.id`
- `offerId` → Affiliate Network → `offers.id` (canonical)
- `testEmailId` → Email Seeder → `test_emails.id`
- `testAccountId` → Email Seeder → `test_accounts.id`

These IDs are passed between services via URLs and events.  
They are the backbone for attribution and LTV calculations.

## Tracking URL Contract

All tracked links (email, push, SMS) use this pattern:

```txt
https://TRK_DOMAIN/click
  ?oid=<OFFER_ID>        # offerId
  &cid=<CONTACT_ID>      # Messaging Core contacts.id
  &mid=<MESSAGE_ID>      # Messaging Core outbound_messages.id
  &ch=<CHANNEL>          # 'email' | 'push' | 'sms'
  &src=<OPTIONAL_SOURCE> # campaign/source code (optional)
```

Example (email):
```txt
https://trk.offers.example/click
  ?oid=456
  &cid=3f6f0d1c-2cde-4c33-ae52-0f7e123abc99
  &mid=7b9d2a3e-9fa6-44e1-9de1-2b7dd9dcdef0
  &ch=email
  &src=welcome_series_1
```

Inside the Affiliate Network click handler, map query params:
- `oid` → `offerId`
- `cid` → `sub1` (contactId)
- `mid` → `sub2` (messageId)
- `ch` → `sub3` (channel)
- `src` → `sub4` (optional campaign/source/creative info)

The `src` parameter can encode creative information to avoid breaking the URL contract:
- `src=welcome_series_1::tmpl=welcome_email_v1::var=B`
- `src=push_daily_deal::var=A`

`sub1–sub4` must propagate from clicks → conversions.

Messaging Core must always include `cid`, `mid`, and `ch` when generating links.

## Services

See per-service docs:
- `services/affiliate-network/README.md`
- `services/domain-steward/README.md`
- `services/messaging-core/README.md`
- `services/creative-generator/README.md`
- `services/offer-creator/README.md`
- `services/email-seeder/README.md`

## Architecture & Contracts

- `docs/architecture.md` – end-to-end data flow & responsibilities.
- `docs/repos-and-domains.md` – repositories, domains, and environments.
- `docs/contracts-ids-and-urls.md` – ID rules & tracking URLs.
- `docs/contracts-events.md` – event formats between services.
- `docs/creative-generator.md` – creative system overview and integration.
- `docs/offer-creator.md` – offer orchestration and blueprint management.
- `docs/email-seeder.md` – test email management and QA workflows.

## How AI Tools Should Use This Repo

1. Start with `docs/architecture.md` to understand the system.
2. Respect the ID contracts and tracking URL format in any new code or docs.
3. When adding or changing behavior, update:
   - The relevant `services/*/README.md`
   - Any impacted contract docs under `docs/contracts-*`
4. Prefer:
   - TypeScript, Node.js, Express
   - Drizzle ORM + Neon Postgres
   - React + Tailwind/shadcn for UIs
   - Zod for input validation

See `ai/README.md` for more AI-specific guidance.