# Offer Creator (Offer Orchestrator)

The Offer Creator is an orchestration service that designs, configures, and syncs offers into the Affiliate Network, and defines how offers are used to monetize signup flows.

It sits *above* the Affiliate Network and works closely with:
- **Affiliate Network** – where offers, clicks, and conversions live.
- **Creative Generator** – which creatives are used for an offer.
- **Domain Steward** – which domains are used for landers and tracking.
- **Messaging Core** – which flows and messages promote the offers.

## Responsibilities

- Define **offer blueprints**:
  - Vertical, geo, payout rules, primary KPI (lead/sale), traffic rules, etc.
- Sync offers into the **Affiliate Network** via API:
  - Create/update real offers and receive `offerId` from the network.
  - Maintain mapping between `offerBlueprintId` and network `offerId`.
- Maintain an **offer catalog**:
  - Includes both internally created offers and offers pulled from the network.
  - Classified by vertical, payout, EPC, risk profile, etc.
- Define **signup and monetization flows**:
  - `offerFlowId` = a flow that ties together:
    - Initial signup / primary offer.
    - Upsell/downsell steps (thank-you page offers, later email/push offers).
  - Describe which offers are used at each step.

## Core IDs

- `offerBlueprintId`
  - ID for the logical offer definition (owned by Offer Creator).
- `offerFlowId`
  - ID for a signup/monetization flow that uses one or more offers.
- `offerId`
  - Canonical offer ID in the Affiliate Network (referenced by `oid` in tracking URLs).

Offer Creator must **not** redefine `offerId`; it maps to it.

## Integration with Affiliate Network

- Outbound:
  - `Offer Creator → Affiliate Network` via APIs such as:
    - `POST /api/offers` (create offer)
    - `PUT /api/offers/:offerId` (update offer)
  - It sends configuration derived from `offerBlueprintId`.
- Inbound:
  - Receives created/updated `offerId` and stores mapping:
    - `offerBlueprintId` ↔ `offerId`.

The Affiliate Network remains the source of truth for:
- Live offer status.
- Payouts, caps, and routing.
- Clicks and conversions tied to `offerId`.

## Integration with Messaging Core

- Offer Creator defines which offers are part of a flow:
  - For a given `offerFlowId`, specify which `offerId`s to promote in:
    - Welcome sequences.
    - Reactivation sequences.
    - Post-signup offers.

Messaging Core:
- Uses `offerFlowId` configuration to pick `offerId` for a given contact and step.
- Builds tracking URLs including `oid=<offerId>` as per `contracts-ids-and-urls.md`.

## Integration with Creative Generator

- For each `offerBlueprintId` and/or `offerId`, Offer Creator:
  - Specifies which creative templates/variants are eligible.
  - Provides context to Creative Generator (vertical, angle, compliance rules).

Creative Generator:
- Produces email, push, and lander creatives for those offers.
- Returns `creativeTemplateId` and `creativeVariantId` that can be attached to flows.

## Integration with Domain Steward

- Offer Creator uses domain roles rather than raw domain strings:
  - `landerDomainRole`, `trackingDomainRole`, etc.
- Domain Steward resolves which actual domains to use for:
  - landers
  - tracking
  - email sending

Offer Creator does not manage DNS; it just references domain roles or IDs.

## Example: Pickleball Offer

- `offerBlueprintId`: `pickleball_signup_v1`
  - Vertical: sports
  - Primary action: free signup
  - Monetization: core offer + post-signup offers

- Offer Creator syncs this as a real offer:
  - Creates offer in Affiliate Network → gets `offerId = 456`.

- `offerFlowId`: `pickleball_onboarding_flow_v1`
  - Step 1: Signup lander for `offerId = 456`.
  - Step 2: Thank-you page offers using other network `offerId`s.
  - Step 3: Welcome email sequence promoting `offerId = 456` and upsells.

## Offer Flow Lifecycle

```txt
1. Design offer blueprint
   ↓
2. Sync to Affiliate Network (get offerId)
   ↓
3. Connect to Creative Generator (get templates)
   ↓
4. Connect to Domain Steward (get domains)
   ↓
5. Define monetization flow (offerFlowId)
   ↓
6. Deploy to Messaging Core (start sending)
   ↓
7. Monitor performance & optimize
```

This orchestration ensures offers are properly configured across all services before going live.