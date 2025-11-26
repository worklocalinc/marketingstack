# Architecture Overview

The WorkLocal Marketing Stack is a performance marketing system composed of three main services:

1. **Affiliate Network Platform**
2. **Domain Steward**
3. **Messaging Core**

## Data Flow (End-to-End)

1. **User visits a lander**
   - Landers live on domains configured by **Domain Steward**.
   - Landers may include:
     - Affiliate tracking links.
     - Web Push subscription prompts.
     - Email capture forms.

2. **Contact creation (Messaging Core)**
   - When a user submits an email or allows push:
     - `Messaging Core` creates/updates a `contact` record.
     - Web Push details are stored in `push_subscriptions`.
     - Channel-specific consent is stored in `contact_channels`.

3. **Outbound messaging (Messaging Core)**
   - For each email or push send:
     - Create an `outbound_messages` row with a new `messageId`.
     - Generate tracking URLs using:
       - `cid` = `contactId`
       - `mid` = `messageId`
       - `ch`  = channel (`email` | `push` | later `sms`)
       - `oid` = offerId (from Affiliate Network)
       - `src` = optional campaign/source code
     - Email is sent via ESP (e.g., Resend). Push is sent via Web Push.

4. **Click tracking (Affiliate Network)**
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

5. **Conversion tracking (Affiliate Network)**
   - When a conversion happens:
     - The Affiliate Network records a `conversion` tied to a `clickId`.
     - SubIDs (`sub1–sub4`) are propagated to the conversion record.
     - Revenue, payout, profit are computed and stored.

6. **Event ingestion (Messaging Core)**
   - The Affiliate Network emits events (e.g. `lead.created`, `sale.created`) to:
     - `Messaging Core` via `POST /api/events/affiliate`.
   - Messaging Core:
     - Uses `contactId` and `messageId` from subIDs to attach revenue back to people and messages.
     - Enables LTV, per-message ROI, and per-channel performance.

7. **Domain management (Domain Steward)**
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