# Creative Generator (Creative Studio)

The Creative Generator (Creative Studio) is the service that creates and manages marketing creatives:
- Email templates and variants
- Web push copy and variants
- Ad and lander copy for offers

It does **not** send messages or track user behavior. It only defines **what** can be sent or shown.

## Responsibilities

- Store reusable templates for:
  - Email: subject, preheader, body, CTAs, layout blocks.
  - Web push: title, body, icon, target URL placeholders.
  - Ad / lander copy: headlines, body sections, bullets, hooks, angles.
- Generate new variants using AI (LLM-based) following:
  - Brand rules
  - Vertical-specific constraints
  - Compliance constraints
- Expose an API for other services to:
  - Fetch available templates/variants for a given offer/vertical.
  - Request new variants for a given template.
  - Log performance feedback (e.g., which variants are performing best).

## Core IDs

- `creativeTemplateId`
  - Primary identifier for a logical template (e.g., "welcome_email_v1").
- `creativeVariantId`
  - Identifier for a specific variant under a template (e.g., "A", "B", "C" or a UUID).
- `creativeSetId` (optional / planned)
  - Group of templates/variants used together for an offer or funnel.

These IDs must be stable so other services can reference them.

## Integration with Messaging Core

Messaging Core owns contacts, consent, and outbound_messages.

- When preparing to send a message:
  1. Messaging Core asks Creative Generator:
     - "Give me an email template/variant for offer X, purpose Y (e.g., welcome, reactivation)."
  2. Creative Generator responds with template + chosen variant:
     - `creativeTemplateId`
     - `creativeVariantId`
     - content blocks (subject/body/CTA URLs placeholders).
  3. Messaging Core:
     - Creates `outbound_messages` row with:
       - `creativeTemplateId`
       - `creativeVariantId`
       - `channel`
     - Builds tracking URLs as defined in `docs/contracts-ids-and-urls.md`.
     - Sends via ESP or Web Push.

- For attribution:
  - `outbound_messages` is the source of truth for which creative was used.
  - `src` in the tracking URL can optionally encode campaign/sequence/creative.

## Integration with Affiliate Network

Affiliate Network owns offers, clicks, conversions.

- Offers may have associated creative sets:
  - Email sequences
  - Ad/lander angles
- Affiliate Network can:
  - Reference `creativeSetId` or `defaultCreativeTemplateId` per offer.
  - Optionally store the creative variant used per click or placement.

## Integration with Domain Steward

Domain Steward controls domains and roles.

- Creative Generator uses generic placeholders for domains (e.g., `{{TRACKING_URL}}`, `{{LANDER_URL}}`).
- Messaging Core and Affiliate Network resolve these placeholders using Domain Steward-configured domains.

## AI Generation (LLM) Expectations

Creative Generator may use one or more AI models to generate variants:

- Input:
  - Vertical, offer description, constraints.
  - Base template or angle.
  - Brand rules (tone, forbidden phrases).
- Output:
  - Draft creative variants with metadata.
- Human or automated filters can approve/reject variants before use.

This service defines the **structure and rules** for generative workflows; the exact model implementation can change without affecting other services as long as the contracts and IDs remain stable.

## Creative Encoding in src Parameter

To avoid breaking the existing URL contract, creative information is encoded in the `src` parameter:

### Examples:

- `src=welcome_series_1::tmpl=welcome_email_v1::var=B`
- `src=push_daily_deal::var=A`
- `src=leadgen_angle3::fb_adset_27`

The Affiliate Network still maps `src` → `sub4`, preserving the existing contract.

## Creative Performance Feedback Loop

```txt
1. Creative Generator creates variant
   ↓
2. Messaging Core sends using variant
   ↓
3. Affiliate Network tracks conversions
   ↓
4. Performance metrics fed back to Creative Generator
   ↓
5. Creative Generator optimizes variant selection
```

This closed loop enables automated creative optimization over time.