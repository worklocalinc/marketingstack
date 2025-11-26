# QA Flow Tester

Test offer flows end-to-end using Email Seeder.

## Services Involved
- **Email Seeder** (`seed.adjump.com`) - Provides test emails
- **Messaging Core** (`messages.adjump.com`) - Registers contacts, sends messages
- **Affiliate Network** (`adjump.com`) - Tracks clicks and conversions
- **Offer Creator** (`houseoffers.adjump.com`) - Defines offer flows

## Test Flow Steps
1. Get test email from Email Seeder
2. Register as contact in Messaging Core
3. Trigger message send (email/push)
4. Check Email Seeder inbox for received message
5. Extract and validate tracking URL
6. Simulate click → verify in Affiliate Network
7. Simulate conversion → verify attribution

## Tracking URL Validation
Expected format:
```
https://adjump.com/click?oid=<offerId>&cid=<contactId>&mid=<messageId>&ch=email&src=<campaign>
```

## Task
Test flow: $ARGUMENTS

Generate test code or curl commands that:
1. Create/get a test email from Email Seeder API
2. POST to Messaging Core to create contact
3. Trigger a message send
4. Query Email Seeder inbox for the message
5. Validate tracking URL format
6. POST click to Affiliate Network
7. POST conversion and verify subIDs propagated

Output a test script or step-by-step commands with expected responses.
