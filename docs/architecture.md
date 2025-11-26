# Architecture Overview

The WorkLocal Marketing Stack is a performance marketing system composed of six main services:

1. **Affiliate Network Platform**
2. **Domain Steward**
3. **Messaging Core**
4. **Creative Generator (Creative Studio)**
5. **Offer Creator (Offer Orchestrator)**
6. **Email Seeder**

## Data Flow (End-to-End)

0. **Offer setup (Offer Creator)**
   - Offer blueprints are designed in **Offer Creator**
   - Synced to **Affiliate Network** to get canonical `offerId`
   - Connected to creatives, domains, and flows

1. **User visits a lander**
   - Landers live on domains configured by **Domain Steward**.
   - Landers may include:
     - Affiliate tracking links (using `offerId` from Offer Creator flow).
     - Web Push subscription prompts.
     - Email capture forms.

2. **Contact creation (Messaging Core)**
   - When a user submits an email or allows push:
     - `Messaging Core` creates/updates a `contact` record.
     - Web Push details are stored in `push_subscriptions`.
     - Channel-specific consent is stored in `contact_channels`.

3. **Creative selection (Creative Generator + Messaging Core)**
   - When preparing to send a message:
     - Messaging Core queries Creative Generator for appropriate template/variant
     - Creative Generator returns:
       - `creativeTemplateId` and `creativeVariantId`
       - Content blocks with placeholders (subject, body, CTAs)
     - Messaging Core records which creative was selected

4. **Outbound messaging (Messaging Core)**
   - For each email or push send:
     - Create an `outbound_messages` row with:
       - New `messageId`
       - `creativeTemplateId` and `creativeVariantId`
     - Generate tracking URLs using:
       - `cid` = `contactId`
       - `mid` = `messageId`
       - `ch`  = channel (`email` | `push` | later `sms`)
       - `oid` = offerId (from Affiliate Network)
       - `src` = optional campaign/source/creative encoding
     - Replace placeholders in creative content
     - Email is sent via ESP (e.g., Resend). Push is sent via Web Push.

5. **Click tracking (Affiliate Network)**
   - The tracking link goes to the Affiliate Network click endpoint:
     - `https://TRK_DOMAIN/click?...`
   - The Affiliate Network:
     - Records a `click` with `clickId`.
     - Stores subIDs:
       - `sub1` = `contactId`
       - `sub2` = `messageId`
       - `sub3` = `channel`
       - `sub4` = optional `src`
     - Redirects to the relevant offer or lander domain.

6. **Conversion tracking (Affiliate Network)**
   - When a conversion happens:
     - The Affiliate Network records a `conversion` tied to a `clickId`.
     - SubIDs (`sub1–sub4`) are propagated to the conversion record.
     - Revenue, payout, profit are computed and stored.

7. **Event ingestion (Messaging Core)**
   - The Affiliate Network emits events (e.g. `lead.created`, `sale.created`) to:
     - `Messaging Core` via `POST /api/events/affiliate`.
   - Messaging Core:
     - Uses `contactId` and `messageId` from subIDs to attach revenue back to people and messages.
     - Enables LTV, per-message ROI, and per-channel performance.

8. **Domain management (Domain Steward)**
   - Domain Steward manages:
     - Domain inventory (owned and rented).
     - DNS configuration via Cloudflare.
     - Domain usage roles: tracking, lander, email, CDN, etc.
   - Other services reference domains via `domain_usages` and `domainId`, not raw strings.

## Service Boundaries

### Messaging Core

- Owns:
  - `contacts`, `contact_channels`
  - `push_subscriptions`
  - `outbound_messages`
- Integrations:
  - ESP (e.g., Resend) for email.
  - Web Push (VAPID config) for push delivery.
  - Incoming Affiliate Network events.

### Affiliate Network Platform

- Owns:
  - `offers`, `affiliates`, `advertisers`
  - `clicks`, `conversions`, `payouts`
- Receives:
  - Traffic via tracking links.
- Emits:
  - Events to Messaging Core and potentially to BI/analytics.

### Domain Steward

- Owns:
  - `domains`, `domain_usages`, and infra metadata.
- Integrations:
  - Registrars, Cloudflare, Neon, AdJump.
- Provides:
  - Which domains are used for:
    - tracking (`trk.*`)
    - landers (`offer.*`, vertical-specific domains)
    - email sending (`mail.*`)

### Creative Generator (Creative Studio)

- Owns:
  - `creative_templates`, `creative_variants`
  - AI generation prompts and workflows
  - Creative performance metrics
- Does NOT:
  - Send messages (that's Messaging Core)
  - Track clicks/conversions (that's Affiliate Network)
- Provides:
  - Templates and variants for all channels (email, push, ads, landers)
  - AI-assisted creative generation
  - A/B testing variants
  - Performance-based variant selection

### Offer Creator (Offer Orchestrator)

- Owns:
  - `offer_blueprints`, `offer_flows`
  - Offer catalog and monetization sequences
  - Blueprint to network synchronization
- Does NOT:
  - Track clicks/conversions (that's Affiliate Network)
  - Own canonical `offerId` (that's Affiliate Network)
  - Send messages (that's Messaging Core)
  - Create content (that's Creative Generator)
- Provides:
  - Offer blueprint design and management
  - Sync to Affiliate Network to create real offers
  - Multi-step monetization flow definitions
  - Offer catalog aggregation from all sources

### Email Seeder

- Owns:
  - `test_accounts`, `test_emails`
  - `inbox_messages`, `qa_test_runs`
  - Managed mailbox access and QA workflows
- Does NOT:
  - Send production emails (that's Messaging Core)
  - Manage real contacts (that's Messaging Core)
  - Track conversions (that's Affiliate Network)
- Provides:
  - Pre-configured test email addresses for QA
  - Inbox monitoring for test verification
  - Automated signup and offer flow testing
  - End-to-end validation before production

## Tech Stack Expectations

- Backend:
  - Node.js + TypeScript
  - Express
  - Drizzle ORM + Neon Postgres
  - Zod for input validation
- Frontend:
  - React + TypeScript
  - Tailwind/shadcn-style UI
  - React Query for data fetching
- Multi-tenant:
  - `organizationId` and/or `networkId` added to key tables for future scaling.