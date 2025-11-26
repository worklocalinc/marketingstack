# AI Usage Guide

This file is written for AI assistants (ChatGPT, GitHub agents, Copilot, etc.) that work on this repo.

## Core Principles

1. **Preserve Architecture Boundaries**

   - **Affiliate Network** = offers, affiliates, clicks, conversions, revenue, payouts.
   - **Domain Steward** = domains, DNS, infra, and domain usage (tracking, lander, email, CDN).
   - **Messaging Core** = contacts, consent, outbound messages, channel-specific delivery (email, push).
   - **Creative Generator** = templates, variants, creative assets, AI generation (does NOT send or track).
   - **Offer Creator** = offer blueprints, flows, catalog, sync to network (orchestration layer).
   - **Email Seeder** = test emails, QA workflows, inbox monitoring (does NOT send production messages).

   Do not mix responsibilities between services.

2. **Respect ID & URL Contracts**

   - Always use the canonical IDs:
     - `contactId`, `messageId`, `subscriptionId`, `clickId`, `conversionId`
     - `creativeTemplateId`, `creativeVariantId`
     - `offerBlueprintId`, `offerFlowId`, `offerId` (canonical in Affiliate Network)
     - `testEmailId`, `testAccountId` (Email Seeder)
   - Tracking URLs must match the format in `docs/contracts-ids-and-urls.md`.
   - In the Affiliate Network, map link query params `oid/cid/mid/ch/src` to `offerId` and `sub1–sub4`.
   - Creative info can be encoded in `src` param (e.g., `src=campaign::tmpl=template_id::var=B`).

3. **Preferred Stack**

   - Backend: Node.js + TypeScript, Express, Drizzle ORM, Neon Postgres.
   - Validation: Zod.
   - Frontend: React + TypeScript, Tailwind/shadcn-style components.
   - Architecture: multi-tenant-ready (e.g. `organizationId`, `networkId` where relevant).

## How to Extend the System

When you (AI) add or change functionality:

1. **Update Docs First**
   - Modify the relevant `services/*/README.md`.
   - If you introduce or change a contract (IDs, URLs, events), update:
     - `docs/contracts-ids-and-urls.md`
     - `docs/contracts-events.md`

2. **Then Add Code**
   - Place backend code under `services/<service-name>/src`.
   - Keep a clear separation:
     - `routes/` – Express route handlers.
     - `schema/` – Drizzle schemas and migrations.
     - `services/` – business logic.
     - `validation/` – Zod schemas.

3. **Document Routing & Data Flow**
   - For each new endpoint:
     - Specify the HTTP method and path.
     - Input and output shape.
     - Which IDs are expected and where they come from.
     - Which other service (if any) is called.

## Safety Rules for This Repo

- Never silently change ID semantics (`cid`, `mid`, `ch`, `sub1–sub4`).
- Do not invent new query params for tracking unless also updating the contract docs.
- Do not collapse services into a single monolith; keep them logically separate.

## Quick Navigation for AI

- Big picture: `docs/architecture.md`
- ID and URL rules: `docs/contracts-ids-and-urls.md`
- Events between services: `docs/contracts-events.md`
- Creative system: `docs/creative-generator.md`
- Offer orchestration: `docs/offer-creator.md`
- Repositories & domains: `docs/repos-and-domains.md`
- Per-service responsibilities:
  - `services/affiliate-network/README.md`
  - `services/domain-steward/README.md`
  - `services/messaging-core/README.md`
  - `services/creative-generator/README.md`
  - `services/offer-creator/README.md`
  - `services/email-seeder/README.md`

## Creative-Specific Guidance

When working on templates, variants, or AI generation:
- Use `services/creative-generator/` and `docs/creative-generator.md` as the source of truth
- Creative IDs (`creativeTemplateId`, `creativeVariantId`) are canonical and must be preserved
- Creative Generator only creates content; it does NOT send or track anything

## Offer-Specific Guidance

When working on offer creation, flows, or catalogs:
- Use `services/offer-creator/` and `docs/offer-creator.md` as the source of truth
- Offer Creator orchestrates but does NOT own the canonical `offerId` (that's Affiliate Network)
- New offers like the Pickleball offer start as `offerBlueprintId` then sync to get `offerId`
- Monetization flows (`offerFlowId`) define multi-step sequences across services

## Email Seeder-Specific Guidance

When working on test emails, QA workflows, or inbox monitoring:
- Use `services/email-seeder/` and `docs/email-seeder.md` as the source of truth
- Email Seeder provides test emails but does NOT send production messages (that's Messaging Core)
- Test IDs (`testEmailId`, `testAccountId`) are used for QA workflows
- QA test runs can verify the full flow: signup → email received → click tracked → conversion recorded
- Test emails should be clearly tagged and never mixed with production contacts