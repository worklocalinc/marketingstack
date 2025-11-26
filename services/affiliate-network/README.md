# Affiliate Network Service

This service owns offers, affiliates, advertisers, clicks, conversions, and payouts.

## Responsibilities

- Receive tracked traffic via `/click` endpoint(s).
- Redirect users to offers or landers.
- Record `clicks` with `clickId` and subIDs (`sub1–sub4`).
- Record `conversions` tied to `clickId`.
- Compute revenue, payout, and profit.
- Emit events (e.g., `lead.created`, `sale.created`) to Messaging Core.

## Core Concepts

- **Offer** – something visitors can sign up or buy.
- **Click** – a tracked visit; contains:
  - `clickId`
  - `offerId`
  - `sub1` = `contactId`
  - `sub2` = `messageId`
  - `sub3` = `channel`
  - `sub4` = optional `src`
- **Conversion** – a tracked completion event from a click.

## HTTP Endpoints (Planned)

- `GET /click`
  - Query: `oid`, `cid`, `mid`, `ch`, `src`
  - Behavior:
    - Create `click` row.
    - Map query params to subIDs.
    - Redirect to appropriate lander/offer URL (using Domain Steward-configured domains).

- `POST /api/conversions`
  - Used by pixels or server-to-server integrations.
  - Body includes `clickId` or equivalent and revenue details.

- `POST /api/events/export`
  - (Planned) For exporting summarized events to other systems.

## Integration with Other Services

- **Domain Steward**:
  - Used to determine the domain/URL that clicks redirect to.
  - Uses domain metadata to choose the best lander/tracking domain.

- **Messaging Core**:
  - Receives events via `/api/events/affiliate` endpoint on Messaging Core.
  - Provides:
    - `contactId` (from `sub1`)
    - `messageId` (from `sub2`)
    - `channel`  (from `sub3`)
    - optional `src` (from `sub4`)
    - revenue and payout info.

## Database Schema (Conceptual)

### Core Tables

```sql
-- Offers
offers:
  - id (offerId)
  - organizationId
  - name
  - description
  - url
  - defaultPayout
  - status
  - vertical
  - timestamps

-- Affiliates
affiliates:
  - id (affiliateId)
  - organizationId
  - name
  - email
  - status
  - defaultPayoutPercentage
  - timestamps

-- Clicks
clicks:
  - clickId (unique)
  - organizationId
  - offerId
  - affiliateId (optional)
  - sub1 (contactId)
  - sub2 (messageId)
  - sub3 (channel)
  - sub4 (source/campaign)
  - ip
  - userAgent
  - referer
  - timestamp

-- Conversions
conversions:
  - id (conversionId)
  - organizationId
  - clickId
  - offerId
  - affiliateId (optional)
  - type (lead/sale)
  - revenue
  - payout
  - profit
  - status
  - sub1 (contactId - propagated from click)
  - sub2 (messageId - propagated from click)
  - sub3 (channel - propagated from click)
  - sub4 (source - propagated from click)
  - timestamp
```

## Environment Variables

```env
# Database
DATABASE_URL=postgresql://...

# Service Authentication
SERVICE_AUTH_TOKEN=...

# Messaging Core Integration
MESSAGING_CORE_URL=https://messaging.example.com
MESSAGING_CORE_TOKEN=...

# Domain Steward Integration
DOMAIN_STEWARD_URL=https://domains.example.com
DOMAIN_STEWARD_TOKEN=...

# Tracking Configuration
DEFAULT_TRACKING_DOMAIN=trk.offers.example
```

## API Examples

### Click Tracking

```http
GET /click?oid=456&cid=3f6f0d1c&mid=7b9d2a3e&ch=email&src=welcome
```

Response: 302 Redirect to offer URL

### Conversion Postback

```http
POST /api/conversions
Content-Type: application/json
Authorization: Bearer <TOKEN>

{
  "clickId": "clk_abc123",
  "type": "lead",
  "revenue": 5.00,
  "status": "approved"
}
```

### Event Export to Messaging Core

```http
POST https://messaging.example.com/api/events/affiliate
Content-Type: application/json
Authorization: Bearer <SERVICE_TOKEN>

{
  "eventType": "lead.created",
  "timestamp": "2024-01-15T10:30:00Z",
  "data": {
    "clickId": "clk_abc123",
    "conversionId": "conv_xyz789",
    "offerId": 456,
    "contactId": "3f6f0d1c-2cde-4c33-ae52-0f7e123abc99",
    "messageId": "7b9d2a3e-9fa6-44e1-9de1-2b7dd9dcdef0",
    "channel": "email",
    "source": "welcome_series_1",
    "revenue": 5.00,
    "payout": 3.50,
    "profit": 1.50
  }
}
```