# Event Contracts

This document defines the event formats and contracts between services.

## Event Flow Overview

```txt
Affiliate Network → Messaging Core → Analytics/BI
```

The primary event flow is from the Affiliate Network to Messaging Core, enabling attribution and LTV calculations.

## Affiliate Network → Messaging Core Events

### Endpoint

`POST /api/events/affiliate`

### Event Types

#### Lead Created Event

```json
{
  "eventType": "lead.created",
  "timestamp": "2024-01-15T10:30:00Z",
  "data": {
    "clickId": "clk_abc123",
    "conversionId": "conv_xyz789",
    "offerId": 456,
    "contactId": "3f6f0d1c-2cde-4c33-ae52-0f7e123abc99",  // from sub1
    "messageId": "7b9d2a3e-9fa6-44e1-9de1-2b7dd9dcdef0",  // from sub2
    "channel": "email",                                     // from sub3
    "source": "welcome_series_1",                          // from sub4 (optional)
    "revenue": 5.00,
    "payout": 3.50,
    "profit": 1.50,
    "metadata": {
      "ip": "192.168.1.1",
      "userAgent": "Mozilla/5.0...",
      "country": "US",
      "state": "CA"
    }
  }
}
```

#### Sale Created Event

```json
{
  "eventType": "sale.created",
  "timestamp": "2024-01-15T10:35:00Z",
  "data": {
    "clickId": "clk_abc123",
    "conversionId": "conv_xyz790",
    "offerId": 456,
    "contactId": "3f6f0d1c-2cde-4c33-ae52-0f7e123abc99",  // from sub1
    "messageId": "7b9d2a3e-9fa6-44e1-9de1-2b7dd9dcdef0",  // from sub2
    "channel": "email",                                     // from sub3
    "source": "welcome_series_1",                          // from sub4 (optional)
    "revenue": 49.99,
    "payout": 25.00,
    "profit": 24.99,
    "metadata": {
      "productId": "prod_123",
      "quantity": 1,
      "currency": "USD"
    }
  }
}
```

#### Click Event (Optional)

```json
{
  "eventType": "click.created",
  "timestamp": "2024-01-15T10:25:00Z",
  "data": {
    "clickId": "clk_abc123",
    "offerId": 456,
    "contactId": "3f6f0d1c-2cde-4c33-ae52-0f7e123abc99",
    "messageId": "7b9d2a3e-9fa6-44e1-9de1-2b7dd9dcdef0",
    "channel": "email",
    "source": "welcome_series_1",
    "metadata": {
      "ip": "192.168.1.1",
      "userAgent": "Mozilla/5.0...",
      "referer": "https://example.com"
    }
  }
}
```

## Messaging Core → External Services (Future)

### Webhook Events

For future integrations with external services (e.g., CRMs, Analytics):

#### Contact Created

```json
{
  "eventType": "contact.created",
  "timestamp": "2024-01-15T09:00:00Z",
  "data": {
    "contactId": "3f6f0d1c-2cde-4c33-ae52-0f7e123abc99",
    "email": "user@example.com",
    "channels": ["email", "push"],
    "metadata": {
      "source": "landing_page",
      "ip": "192.168.1.1"
    }
  }
}
```

#### Message Sent

```json
{
  "eventType": "message.sent",
  "timestamp": "2024-01-15T10:20:00Z",
  "data": {
    "messageId": "7b9d2a3e-9fa6-44e1-9de1-2b7dd9dcdef0",
    "contactId": "3f6f0d1c-2cde-4c33-ae52-0f7e123abc99",
    "channel": "email",
    "templateId": "welcome_email_v1",
    "providerMessageId": "resend_msg_123",
    "metadata": {
      "campaignId": "welcome_series",
      "sequenceStep": 1
    }
  }
}
```

## Domain Steward Events (Future)

### Domain Status Changed

```json
{
  "eventType": "domain.status_changed",
  "timestamp": "2024-01-15T08:00:00Z",
  "data": {
    "domainId": "dom_123",
    "domainName": "trk.example.com",
    "previousStatus": "pending",
    "newStatus": "active",
    "metadata": {
      "registrar": "namesilo",
      "dnsProvider": "cloudflare"
    }
  }
}
```

## Event Processing Guidelines

### For Event Producers

1. Always include required IDs (`contactId`, `messageId`, etc.)
2. Use ISO 8601 timestamps with timezone
3. Include relevant metadata for debugging
4. Ensure idempotency keys for critical events

### For Event Consumers

1. Validate event schema before processing
2. Handle duplicates gracefully (idempotency)
3. Log failed events for retry
4. Process events asynchronously when possible

## Authentication

Events between services should include authentication:

```http
POST /api/events/affiliate
Authorization: Bearer <SERVICE_TOKEN>
Content-Type: application/json
```

Where `SERVICE_TOKEN` is a shared secret or JWT token configured per environment.