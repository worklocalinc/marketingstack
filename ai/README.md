# AI Usage Guide

This file is written for AI assistants (ChatGPT, GitHub agents, Copilot, etc.) that work on this repo.

## Core Principles

1. **Preserve Architecture Boundaries**

   - **Affiliate Network** = offers, affiliates, clicks, conversions, revenue, payouts.
   - **Domain Steward** = domains, DNS, infra, and domain usage (tracking, lander, email, CDN).
   - **Messaging Core** = contacts, consent, outbound messages, channel-specific delivery (email, push).

   Do not mix responsibilities between services.

2. **Respect ID & URL Contracts**

   - Always use the canonical IDs:
     - `contactId`, `messageId`, `subscriptionId`, `clickId`, `conversionId`.
   - Tracking URLs must match the format in `docs/contracts-ids-and-urls.md`.
   - In the Affiliate Network, map link query params `oid/cid/mid/ch/src` to `offerId` and `sub1–sub4`.

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
- Per-service responsibilities:
  - `services/affiliate-network/README.md`
  - `services/domain-steward/README.md`
  - `services/messaging-core/README.md`