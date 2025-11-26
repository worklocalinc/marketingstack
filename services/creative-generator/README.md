# Creative Generator Service (Creative Studio)

This service manages marketing creatives (templates and variants) and provides AI-assisted generation workflows.

## Responsibilities

- Store and version email, push, and ad/lander templates.
- Manage variants for testing (A/B/C, multivariate).
- Provide an API for:
  - Fetching templates/variants by purpose, offer, vertical.
  - Generating new variants using AI.
  - Recording performance signals (CTR, CVR) per variant.

## NOT Responsibilities

- Sending messages (owned by Messaging Core).
- Tracking clicks or conversions (owned by Affiliate Network).
- Managing domains or DNS (owned by Domain Steward).

## Core Concepts

- **Creative Template** – The base structure and rules for a creative type
- **Creative Variant** – A specific version of a template for testing
- **Creative Set** – A collection of related templates for a campaign/funnel

## HTTP Endpoints (Planned)

### Template Management

#### `POST /api/templates/query`

Request a template/variant for a use case.

**Request:**
```json
{
  "channel": "email",
  "purpose": "welcome",
  "offerId": 456,
  "vertical": "finance",
  "organizationId": "org_123",
  "networkId": "net_abc"
}
```

**Response:**
```json
{
  "creativeTemplateId": "welcome_email_v1",
  "creativeVariantId": "welcome_email_v1_B",
  "channel": "email",
  "structure": {
    "subject": "Welcome to XYZ – your next step in {{vertical}}",
    "preheader": "Let's get you set up in a minute.",
    "bodyHtml": "<p>...</p>",
    "ctaLabel": "Get started",
    "ctaUrlPlaceholder": "{{TRACKING_URL}}"
  },
  "metadata": {
    "vertical": "finance",
    "offerId": 456,
    "purpose": "welcome"
  }
}
```

#### `POST /api/templates/generate`

Ask the service to create new variants (AI-assisted).

**Request:**
```json
{
  "baseTemplateId": "welcome_email_v1",
  "numVariants": 3,
  "constraints": {
    "tone": "friendly",
    "maxSubjectLength": 60,
    "forbiddenPhrases": ["guaranteed", "risk-free"]
  }
}
```

#### `GET /api/templates/:id`

Get template details and all variants.

#### `PUT /api/templates/:id/variants/:variantId`

Update a specific variant.

### Performance Tracking

#### `POST /api/performance/record`

Record performance metrics for a variant.

**Request:**
```json
{
  "creativeTemplateId": "welcome_email_v1",
  "creativeVariantId": "welcome_email_v1_B",
  "stats": {
    "impressions": 1000,
    "clicks": 120,
    "conversions": 15
  }
}
```

#### `GET /api/performance/report`

Get performance report for templates/variants.

## Database Schema (Conceptual)

```sql
-- Creative Templates
creative_templates:
  - id (creativeTemplateId)
  - organizationId
  - networkId
  - channel (email/push/lander/ad)
  - purpose (welcome/reactivation/promo/etc)
  - vertical
  - name
  - description
  - baseStructure (JSON)
  - constraints (JSON)
  - status (active/paused/draft)
  - timestamps

-- Creative Variants
creative_variants:
  - id (creativeVariantId)
  - templateId
  - version
  - name (A/B/C or descriptive)
  - status (active/paused/draft/winner/loser)
  - content (JSON - rendered content blocks)
  - metadata (JSON)
  - performance (JSON - CTR, CVR, etc.)
  - generatedBy (human/ai/hybrid)
  - timestamps

-- Creative Sets
creative_sets:
  - id (creativeSetId)
  - organizationId
  - name
  - description
  - templates (JSON array of templateIds)
  - offerIds (JSON array)
  - vertical
  - status
  - timestamps

-- Generation Jobs
generation_jobs:
  - id
  - templateId
  - status (pending/processing/completed/failed)
  - input (JSON - prompts, constraints)
  - output (JSON - generated variants)
  - model
  - timestamps
```

## Environment Variables

```env
# Database
DATABASE_URL=postgresql://...

# AI/LLM Configuration
OPENAI_API_KEY=...
ANTHROPIC_API_KEY=...
AI_MODEL_PREFERENCE=gpt-4

# Service Authentication
SERVICE_AUTH_TOKEN=...

# Messaging Core Integration
MESSAGING_CORE_URL=https://messaging.example.com
MESSAGING_CORE_TOKEN=...

# Content Safety
CONTENT_MODERATION_ENABLED=true
FORBIDDEN_PHRASES_LIST=...

# Performance Thresholds
MIN_CTR_THRESHOLD=0.02
MIN_CVR_THRESHOLD=0.005
```

## Integration Examples

### Messaging Core Requesting a Template

```http
POST /api/templates/query
Content-Type: application/json
Authorization: Bearer <SERVICE_TOKEN>

{
  "channel": "email",
  "purpose": "reactivation",
  "offerId": 789,
  "vertical": "health",
  "contactSegment": "30_day_inactive"
}
```

### AI Generation Request

```http
POST /api/templates/generate
Content-Type: application/json
Authorization: Bearer <SERVICE_TOKEN>

{
  "baseTemplateId": "reactivation_email_v2",
  "numVariants": 5,
  "constraints": {
    "tone": "urgent_but_friendly",
    "maxSubjectLength": 50,
    "includeKeywords": ["limited time", "exclusive"],
    "excludeKeywords": ["spam", "free money"],
    "complianceLevel": "strict"
  },
  "testingStrategy": "multivariate"
}
```

## Creative Placeholders

Templates use placeholders that get replaced by consuming services:

- `{{TRACKING_URL}}` - Replaced by Messaging Core with tracking link
- `{{LANDER_URL}}` - Replaced by Affiliate Network with lander link
- `{{UNSUBSCRIBE_URL}}` - Replaced by Messaging Core
- `{{CONTACT_NAME}}` - Replaced with contact's name
- `{{OFFER_NAME}}` - Replaced with offer name
- `{{VERTICAL}}` - Replaced with vertical name

## Performance Optimization

The service tracks performance metrics and can:

1. Auto-pause underperforming variants
2. Promote winning variants
3. Generate new variants based on winners
4. Provide recommendations for improvement

## Compliance and Safety

- Content moderation for generated text
- Forbidden phrase detection
- Vertical-specific compliance rules
- Brand safety checks
- Approval workflows for AI-generated content