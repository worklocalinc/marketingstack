# Offer Creator Service (Offer Orchestrator)

This service defines and manages **offer blueprints** and **offer flows**, and syncs them into the Affiliate Network.

## Responsibilities

- Create and manage `offerBlueprintId`s:
  - Logical offers, such as the Pickleball signup offer.
- Sync offers into the Affiliate Network:
  - Call network APIs to create/update real `offerId`s.
- Maintain an **offer catalog** combining:
  - Internally created offers (from blueprints).
  - Offers imported from the Affiliate Network.
- Define `offerFlowId`s:
  - Signup/monetization flows that describe:
    - Which offer is shown at signup.
    - Which offers appear on thank-you / upsell pages.
    - Which offers are promoted in email/push sequences.

## NOT Responsibilities

- Tracking or attribution (Affiliate Network).
- Managing contacts or messages (Messaging Core).
- Domains and DNS (Domain Steward).
- Creative content (Creative Generator).

## Core Concepts

- **Offer Blueprint** – The logical design of an offer before it exists in any network
- **Offer Flow** – A multi-step monetization sequence using one or more offers
- **Offer Catalog** – Consolidated view of all available offers from all sources

## HTTP Endpoints (Planned)

### Blueprint Management

#### `POST /api/offer-blueprints`

Create a logical offer blueprint.

**Request:**
```json
{
  "name": "Pickleball Signup Offer v1",
  "offerBlueprintId": "pickleball_signup_v1",
  "vertical": "sports",
  "primaryAction": "lead",
  "payoutModel": "cpl",
  "targetGeo": ["US"],
  "organizationId": "org_123",
  "networkId": "net_abc",
  "rules": {
    "allowedTraffic": ["email", "push"],
    "disallowedTraffic": ["incentivized"],
    "caps": null
  }
}
```

#### `POST /api/offer-blueprints/:offerBlueprintId/sync`

Sync a blueprint into the Affiliate Network and store the mapping.

- Calls Affiliate Network: `POST /api/offers` or `PUT /api/offers/:offerId`
- On success, persists `offerId` for this `offerBlueprintId`

#### `GET /api/offer-blueprints`

List all offer blueprints with their sync status.

### Offer Catalog

#### `GET /api/offers/catalog`

Return a consolidated catalog of offers for use by UIs or decision engines.

**Response:**
```json
{
  "offers": [
    {
      "offerId": 456,
      "offerBlueprintId": "pickleball_signup_v1",
      "name": "Pickleball Signup Offer v1",
      "vertical": "sports",
      "status": "active",
      "payoutModel": "cpl",
      "epc": 2.50,
      "conversionRate": 0.15
    }
  ]
}
```

### Flow Management

#### `POST /api/offer-flows`

Create or update an `offerFlowId` (signup/monetization flow).

**Request:**
```json
{
  "offerFlowId": "pickleball_onboarding_flow_v1",
  "name": "Pickleball Onboarding Flow v1",
  "primaryOfferId": 456,
  "steps": [
    {
      "stepType": "signup_lander",
      "offerId": 456
    },
    {
      "stepType": "thank_you_page",
      "offerId": 789
    },
    {
      "stepType": "email_sequence",
      "offerId": 456,
      "sequenceKey": "welcome_series_1"
    }
  ]
}
```

#### `GET /api/offer-flows/:offerFlowId`

Get details of a specific offer flow including performance metrics.

#### `GET /api/offer-flows/:offerFlowId/next-offer`

Get the next offer in a flow based on current step and context.

**Request:**
```json
{
  "contactId": "contact_123",
  "currentStep": "signup_complete",
  "context": {
    "source": "email",
    "geo": "US"
  }
}
```

## Database Schema (Conceptual)

```sql
-- Offer Blueprints
offer_blueprints:
  - offerBlueprintId
  - organizationId
  - networkId
  - name
  - description
  - vertical
  - primaryAction (lead/sale)
  - payoutModel (cpl/cpa/revshare)
  - targetGeo (JSON array)
  - rules (JSON)
  - status (draft/active/paused)
  - timestamps

-- Blueprint to Network Mapping
offer_blueprint_network_map:
  - offerBlueprintId
  - offerId (from Affiliate Network)
  - networkId
  - syncStatus (pending/synced/failed)
  - lastSyncAt
  - timestamps

-- Offer Flows
offer_flows:
  - offerFlowId
  - organizationId
  - networkId
  - name
  - description
  - primaryOfferId
  - status (draft/active/paused)
  - performance (JSON - conversion rates, revenue)
  - timestamps

-- Offer Flow Steps
offer_flow_steps:
  - id
  - offerFlowId
  - stepOrder
  - stepType (signup_lander/thank_you_page/email_sequence/push_sequence)
  - offerId
  - configuration (JSON - timing, conditions)
  - performance (JSON - step-specific metrics)
  - timestamps

-- Offer Catalog (Materialized View)
offer_catalog:
  - offerId
  - offerBlueprintId (if applicable)
  - source (internal/network/external)
  - name
  - vertical
  - geo
  - payoutModel
  - epc
  - conversionRate
  - status
  - metadata (JSON)
```

## Environment Variables

```env
# Database
DATABASE_URL=postgresql://...

# Service Authentication
SERVICE_AUTH_TOKEN=...

# Affiliate Network Integration
AFFILIATE_NETWORK_URL=https://affiliate.example.com
AFFILIATE_NETWORK_TOKEN=...

# Creative Generator Integration
CREATIVE_GENERATOR_URL=https://creative.example.com
CREATIVE_GENERATOR_TOKEN=...

# Domain Steward Integration
DOMAIN_STEWARD_URL=https://domains.example.com
DOMAIN_STEWARD_TOKEN=...

# Messaging Core Integration
MESSAGING_CORE_URL=https://messaging.example.com
MESSAGING_CORE_TOKEN=...

# Sync Configuration
AUTO_SYNC_ENABLED=true
SYNC_INTERVAL_MINUTES=15
```

## Integration Examples

### Creating and Syncing a Blueprint

```http
# Step 1: Create blueprint
POST /api/offer-blueprints
Content-Type: application/json
Authorization: Bearer <SERVICE_TOKEN>

{
  "offerBlueprintId": "yoga_classes_v1",
  "name": "Online Yoga Classes",
  "vertical": "health_fitness",
  "primaryAction": "lead",
  "payoutModel": "cpl",
  "targetGeo": ["US", "CA"]
}

# Step 2: Sync to network
POST /api/offer-blueprints/yoga_classes_v1/sync
Authorization: Bearer <SERVICE_TOKEN>

# Response includes the network offerId
{
  "success": true,
  "offerId": 789,
  "message": "Offer synced to Affiliate Network"
}
```

### Defining a Monetization Flow

```http
POST /api/offer-flows
Content-Type: application/json
Authorization: Bearer <SERVICE_TOKEN>

{
  "offerFlowId": "yoga_onboarding_v1",
  "name": "Yoga Class Onboarding",
  "primaryOfferId": 789,
  "steps": [
    {
      "stepType": "signup_lander",
      "offerId": 789,
      "configuration": {
        "template": "yoga_landing_v1"
      }
    },
    {
      "stepType": "thank_you_page",
      "offerId": 123,
      "configuration": {
        "delay": 0,
        "condition": "signup_complete"
      }
    },
    {
      "stepType": "email_sequence",
      "offerId": 789,
      "configuration": {
        "sequenceKey": "yoga_welcome",
        "delays": [0, 1440, 4320]
      }
    }
  ]
}
```

## Offer Performance Tracking

The service tracks key metrics for optimization:

- **Blueprint Level**: Overall performance across all uses
- **Flow Level**: Conversion through the entire flow
- **Step Level**: Performance at each step in a flow

These metrics inform:
- Which offers to promote more heavily
- Which flows convert best
- Where users drop off in multi-step flows

## Multi-Network Support (Future)

While initially focused on the internal Affiliate Network, the Offer Creator is designed to support:

- Multiple internal networks
- External affiliate networks (via API adapters)
- Direct advertiser integrations
- Programmatic ad platforms

The `offerBlueprintId` remains constant while `offerId` mappings exist per network.