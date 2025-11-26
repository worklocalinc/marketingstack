# Contract Validator

Validate that code follows the WorkLocal Marketing Stack contracts.

## Canonical IDs (must be preserved end-to-end)
- `contactId` → Messaging Core → `contacts.id`
- `messageId` → Messaging Core → `outbound_messages.id`
- `subscriptionId` → Messaging Core → `push_subscriptions.id`
- `clickId` → Affiliate Network → `clicks.clickId`
- `conversionId` → Affiliate Network → `conversions.id`
- `creativeTemplateId` → Creative Generator → `creative_templates.id`
- `creativeVariantId` → Creative Generator → `creative_variants.id`
- `offerBlueprintId` → Offer Creator → `offer_blueprints.id`
- `offerFlowId` → Offer Creator → `offer_flows.id`
- `offerId` → Affiliate Network → `offers.id`
- `testEmailId` → Email Seeder → `test_emails.id`
- `testAccountId` → Email Seeder → `test_accounts.id`

## Tracking URL Contract
```
https://adjump.com/click
  ?oid=<OFFER_ID>
  &cid=<CONTACT_ID>
  &mid=<MESSAGE_ID>
  &ch=<CHANNEL>          # 'email' | 'push' | 'sms'
  &src=<OPTIONAL_SOURCE>
```

Affiliate Network maps:
- `oid` → `offerId`
- `cid` → `sub1` (contactId)
- `mid` → `sub2` (messageId)
- `ch` → `sub3` (channel)
- `src` → `sub4` (optional)

## Task
Validate: $ARGUMENTS

Search the codebase for:
1. ID usage - Are canonical IDs being used correctly?
2. URL construction - Do tracking URLs match the contract?
3. Event schemas - Do events include required fields?
4. SubID mapping - Is `sub1-sub4` preserved through conversions?

Report any violations with file:line references.
