# Messaging Core Service

This service owns contacts, consent, and outbound messages across channels (email, web push, SMS later).

## Responsibilities

- Store contacts (`contacts`) and channel-specific consent (`contact_channels`).
- Manage web push subscriptions (`push_subscriptions`).
- Create and track outbound messages (`outbound_messages`).
- Integrate with ESP (Resend) for email delivery.
- Integrate with Web Push for browser notifications.
- Ingest events from Affiliate Network for attribution and LTV.

## Core Tables (Conceptual)

```sql
-- Contacts
contacts:
  - id (contactId)
  - organizationId
  - email
  - firstName
  - lastName
  - phone
  - metadata (JSON)
  - ltv (lifetime value)
  - firstSeenAt
  - lastSeenAt
  - timestamps

-- Contact Channels
contact_channels:
  - id
  - contactId
  - channel (email/push/sms)
  - status (subscribed/unsubscribed/bounced/complained)
  - subscribedAt
  - unsubscribedAt
  - metadata (JSON)
  - timestamps

-- Push Subscriptions
push_subscriptions:
  - id (subscriptionId)
  - contactId
  - endpoint
  - p256dh
  - auth
  - browser
  - userAgent
  - isActive
  - lastUsedAt
  - timestamps

-- Outbound Messages
outbound_messages:
  - id (messageId)
  - organizationId
  - contactId
  - channel (email/push/sms)
  - templateId
  - subject
  - content
  - status (queued/sent/failed/bounced/opened/clicked)
  - providerMessageId
  - sentAt
  - openedAt
  - clickedAt
  - metadata (JSON)
  - timestamps

-- Message Templates
message_templates:
  - id (templateId)
  - organizationId
  - name
  - channel
  - subject
  - htmlContent
  - textContent
  - metadata (JSON)
  - timestamps

-- Campaign Performance
campaign_performance:
  - id
  - messageId
  - clickId
  - conversionId
  - revenue
  - payout
  - profit
  - timestamps
```

## Tracking URL Generation

When sending a message, Messaging Core must:

1. Create `outbound_messages` row → get `messageId`.
2. Use Domain Steward to get `TRK_DOMAIN`.
3. Build tracking URL:

```txt
https://TRK_DOMAIN/click
  ?oid=<OFFER_ID>
  &cid=<CONTACT_ID>
  &mid=<MESSAGE_ID>
  &ch=<CHANNEL>
  &src=<OPTIONAL_SOURCE>
```

4. Inject the tracking URL into:
   - Email templates (for CTAs).
   - Push payload (e.g., `url` field used by service worker).

## HTTP Endpoints (Planned)

### Contact Management

- `GET /api/contacts`
  - List contacts with filters

- `POST /api/contacts`
  - Create or update contact

- `GET /api/contacts/:id`
  - Get contact details

- `PUT /api/contacts/:id`
  - Update contact

### Subscription Management

- `POST /api/subscriptions/push`
  - Register web push subscription

- `DELETE /api/subscriptions/push/:id`
  - Remove push subscription

- `POST /api/subscriptions/email`
  - Subscribe email

- `POST /api/unsubscribe`
  - Handle unsubscribe requests

### Message Sending

- `POST /api/messages/send`
  - Send message to contact(s)

- `GET /api/messages`
  - List sent messages

- `GET /api/messages/:id`
  - Get message details

### Event Ingestion

- `POST /api/events/affiliate`
  - Receive events from Affiliate Network

- `POST /api/events/esp`
  - Receive webhooks from ESP (Resend)

### Analytics

- `GET /api/analytics/contacts/:id`
  - Get contact performance metrics

- `GET /api/analytics/campaigns`
  - Get campaign performance data

## Environment Variables

```env
# Database
DATABASE_URL=postgresql://...

# Session
SESSION_SECRET=...

# Email Service (Resend)
RESEND_API_KEY=...
RESEND_FROM_ADDRESS=noreply@example.com

# Web Push (VAPID)
VAPID_PUBLIC_KEY=...
VAPID_SECRET_KEY=...
VAPID_SUBJECT=mailto:admin@example.com

# Service Authentication
MESSAGING_INTERNAL_TOKEN=...

# Domain Steward Integration
DOMAIN_STEWARD_URL=https://domains.example.com
DOMAIN_STEWARD_TOKEN=...

# Default Tracking Domain
DEFAULT_TRACKING_DOMAIN=trk.offers.example
```

## API Examples

### Register Push Subscription

```http
POST /api/subscriptions/push
Content-Type: application/json
Authorization: Bearer <TOKEN>

{
  "contactId": "3f6f0d1c-2cde-4c33-ae52-0f7e123abc99",
  "subscription": {
    "endpoint": "https://fcm.googleapis.com/...",
    "keys": {
      "p256dh": "...",
      "auth": "..."
    }
  },
  "browser": "Chrome",
  "userAgent": "Mozilla/5.0..."
}
```

### Send Email Message

```http
POST /api/messages/send
Content-Type: application/json
Authorization: Bearer <TOKEN>

{
  "contactId": "3f6f0d1c-2cde-4c33-ae52-0f7e123abc99",
  "channel": "email",
  "templateId": "welcome_email_v1",
  "offerId": 456,
  "source": "welcome_series_1",
  "variables": {
    "firstName": "John",
    "offerName": "Special Deal"
  }
}
```

### Event Ingestion from Affiliate Network

```http
POST /api/events/affiliate
Content-Type: application/json
Authorization: Bearer <MESSAGING_INTERNAL_TOKEN>

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

Response:
```json
{
  "success": true,
  "message": "Event processed",
  "data": {
    "contactLTV": 55.00,
    "messageROI": 2.5
  }
}
```

## Push Delivery (Web Push)

Uses VAPID configuration:

- `VAPID_PUBLIC_KEY` - Public key for client-side
- `VAPID_PRIVATE_KEY` - Private key for server-side
- `VAPID_SUBJECT` - Contact email

For each push send:

1. Create `outbound_messages` with `channel='push'`.
2. Fan out to all active `push_subscriptions` for the contact.
3. Include the tracking URL in the payload so the service worker opens it on click.

Example push payload:

```json
{
  "title": "Special Offer!",
  "body": "Get 50% off today only",
  "icon": "/icon-192x192.png",
  "badge": "/badge-72x72.png",
  "url": "https://trk.offers.example/click?oid=456&cid=3f6f0d1c&mid=8213a927&ch=push&src=deal_alert",
  "data": {
    "messageId": "8213a927-1b8a-4c88-a926-9b8122f19c11"
  }
}
```

## Email Delivery (ESP Integration)

Currently using Resend for email delivery:

1. Create `outbound_messages` with `channel='email'`.
2. Generate tracking URLs for all links in the email.
3. Send via Resend API.
4. Store `providerMessageId` from Resend response.
5. Process webhook events (opens, clicks, bounces, complaints).