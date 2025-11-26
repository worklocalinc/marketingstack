# ID and URL Contracts

This document defines the canonical ID and URL rules for the system.  
Any code changes that affect these contracts MUST be reflected here.

## Canonical IDs

- `contactId` → Messaging Core → `contacts.id`
- `messageId` → Messaging Core → `outbound_messages.id`
- `subscriptionId` → Messaging Core → `push_subscriptions.id`
- `clickId` → Affiliate Network → `clicks.clickId`
- `conversionId` → Affiliate Network → `conversions.id`

### Multi-Tenancy IDs (Planned)

- `organizationId` – identity for the owning organization.
- `networkId` – identity for the affiliate network instance (if multiple networks exist).

These should be added to key tables as we implement them.

## Tracking URL Format

Base pattern:

```txt
https://TRK_DOMAIN/click
  ?oid=<OFFER_ID>        # offerId
  &cid=<CONTACT_ID>      # Messaging Core contacts.id
  &mid=<MESSAGE_ID>      # Messaging Core outbound_messages.id
  &ch=<CHANNEL>          # 'email' | 'push' | 'sms'
  &src=<OPTIONAL_SOURCE> # campaign/source code (optional)
```

### Example – Email

```txt
https://trk.offers.example/click
  ?oid=456
  &cid=3f6f0d1c-2cde-4c33-ae52-0f7e123abc99
  &mid=7b9d2a3e-9fa6-44e1-9de1-2b7dd9dcdef0
  &ch=email
  &src=welcome_series_1
```

### Example – Web Push

```txt
https://trk.offers.example/click
  ?oid=456
  &cid=3f6f0d1c-2cde-4c33-ae52-0f7e123abc99
  &mid=8213a927-1b8a-4c88-a926-9b8122f19c11
  &ch=push
  &src=deal_alert_morning
```

## Mapping in Affiliate Network

In the Affiliate Network's click handler:

- `oid` → `offerId`
- `cid` → `sub1` (contactId)
- `mid` → `sub2` (messageId)
- `ch` → `sub3` (channel)
- `src` → `sub4` (optional source/campaign)

These subIDs must flow:

- From clicks → conversions.
- Into events emitted to Messaging Core.

## Messaging Core Responsibilities

- Always set `cid`, `mid`, and `ch` on tracking URLs.
- Include `oid` when a specific offer is associated with the message.
- Optionally include `src` for campaign or sequence identifiers.

## Domain Steward Responsibilities

- Maintain which domain(s) are allowed/assigned as `TRK_DOMAIN`.
- Expose domain usage metadata:
  - e.g., `trk.example.com` tagged as `role = 'tracking'`.