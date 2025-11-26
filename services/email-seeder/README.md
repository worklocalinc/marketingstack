# Email Seeder

The **Email Seeder** service manages pre-configured test email accounts for QA and validation workflows across the marketing stack.

## Purpose

- Provide ready-to-use email addresses for testing signups, offers, and automations
- Validate that email delivery, tracking, and conversion flows work correctly
- Support automated QA pipelines without burning real contacts
- Maintain clean separation between test and production data

## What Email Seeder Owns

- **Test Accounts** – Managed mailboxes for receiving test emails
- **Test Emails** – Pre-configured email addresses with metadata
- **Inbox Access** – API to read incoming emails for verification
- **QA Workflows** – Automated test scenarios for offer validation

## What Email Seeder Does NOT Own

- Sending production emails (that's **Messaging Core**)
- Managing real contacts (that's **Messaging Core**)
- Tracking conversions (that's **Affiliate Network**)
- Domain DNS configuration (that's **Domain Steward**)

## Core Concepts

### Test Account

A managed mailbox that can receive emails. Each account has:
- `testAccountId` – Unique identifier
- `provider` – Email provider (e.g., Gmail, custom IMAP)
- `credentials` – Secure access credentials
- `status` – active, paused, expired

### Test Email

A specific email address within a test account:
- `testEmailId` – Unique identifier
- `testAccountId` – Parent account
- `email` – The actual email address
- `purpose` – signup, offer_test, automation, etc.
- `verticalTags` – job, finance, health, etc.
- `lastUsed` – Timestamp of last use

## HTTP Endpoints (Planned)

### Test Email Management

```
GET    /api/test-emails              # List available test emails
GET    /api/test-emails/:id          # Get specific test email
POST   /api/test-emails              # Create new test email
PUT    /api/test-emails/:id          # Update test email
DELETE /api/test-emails/:id          # Archive test email
```

### Test Account Management

```
GET    /api/test-accounts            # List test accounts
GET    /api/test-accounts/:id        # Get specific account
POST   /api/test-accounts            # Create new account
PUT    /api/test-accounts/:id        # Update account
DELETE /api/test-accounts/:id        # Archive account
```

### Inbox Operations

```
GET    /api/test-emails/:id/inbox              # Get recent emails in inbox
GET    /api/test-emails/:id/inbox/:messageId   # Get specific email content
POST   /api/test-emails/:id/inbox/search       # Search inbox by subject/sender
DELETE /api/test-emails/:id/inbox              # Clear inbox
```

### QA Workflows

```
POST   /api/qa/signup-test           # Run signup flow with test email
POST   /api/qa/offer-test            # Test offer delivery and tracking
POST   /api/qa/automation-test       # Test automation sequence
GET    /api/qa/results/:testId       # Get test results
```

## Conceptual Database Schema

```sql
-- Test accounts (mailboxes)
CREATE TABLE test_accounts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  provider TEXT NOT NULL,  -- 'gmail', 'imap', 'mailinator', etc.
  credentials JSONB NOT NULL,  -- Encrypted
  status TEXT DEFAULT 'active',
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now()
);

-- Test emails within accounts
CREATE TABLE test_emails (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  test_account_id UUID REFERENCES test_accounts(id),
  email TEXT NOT NULL UNIQUE,
  purpose TEXT DEFAULT 'general',  -- 'signup', 'offer_test', 'automation'
  vertical_tags TEXT[],
  metadata JSONB DEFAULT '{}',
  last_used_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- QA test runs
CREATE TABLE qa_test_runs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  test_type TEXT NOT NULL,  -- 'signup', 'offer', 'automation'
  test_email_id UUID REFERENCES test_emails(id),
  offer_id TEXT,
  status TEXT DEFAULT 'pending',  -- 'pending', 'running', 'passed', 'failed'
  results JSONB DEFAULT '{}',
  started_at TIMESTAMPTZ,
  completed_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT now()
);

-- Captured inbox messages
CREATE TABLE inbox_messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  test_email_id UUID REFERENCES test_emails(id),
  from_address TEXT,
  subject TEXT,
  body_text TEXT,
  body_html TEXT,
  headers JSONB,
  received_at TIMESTAMPTZ DEFAULT now()
);
```

## Environment Variables

```bash
# Database
DATABASE_URL=postgres://...

# Email Provider Configs
GMAIL_CLIENT_ID=...
GMAIL_CLIENT_SECRET=...
IMAP_DEFAULT_HOST=...
IMAP_DEFAULT_PORT=993

# Service URLs
MESSAGING_CORE_URL=https://msg.worklocal.dev
AFFILIATE_NETWORK_URL=https://trk.worklocal.dev

# API Security
API_KEY=...
ENCRYPTION_KEY=...  # For credential storage
```

## Integration with Other Services

### With Messaging Core

Email Seeder provides test emails that can be used as contacts:

```
POST /api/contacts
{
  "email": "test-001@seeder.worklocal.dev",
  "source": "email_seeder",
  "testEmailId": "abc123"
}
```

### With Offer Creator

QA workflows can test offer flows end-to-end:

```
POST /api/qa/offer-test
{
  "offerBlueprintId": "pickleball_signup_v1",
  "testEmailId": "test-001-id",
  "validateSteps": ["signup", "email_received", "click_tracked", "conversion"]
}
```

### With Affiliate Network

Verify that test signups properly track through the affiliate system:

```
GET /api/qa/results/:testId
{
  "testId": "qa-run-123",
  "status": "passed",
  "results": {
    "signupCompleted": true,
    "emailReceived": true,
    "clickTracked": true,
    "conversionRecorded": true,
    "clickId": "clk_abc123",
    "conversionId": "conv_xyz789"
  }
}
```

## Use Cases

### 1. Offer Testing

Before launching a new offer, run automated tests:
1. Get available test email from Email Seeder
2. Complete signup flow on offer lander
3. Wait for welcome email to arrive
4. Click tracking link in email
5. Verify click and conversion recorded in Affiliate Network

### 2. Automation Validation

Test that email sequences work correctly:
1. Register test email as contact in Messaging Core
2. Trigger automation sequence
3. Monitor inbox for expected emails
4. Verify timing, content, and tracking

### 3. Pre-Production QA

Run comprehensive test suite before deployments:
1. Test all active offers with fresh test emails
2. Validate tracking URLs resolve correctly
3. Confirm conversions attribute properly
4. Check email rendering across providers

## Best Practices

1. **Rotate test emails** – Don't reuse the same email too frequently
2. **Tag by vertical** – Use vertical-specific test emails for realistic testing
3. **Clean up after tests** – Clear inboxes after test runs
4. **Monitor provider limits** – Stay within email provider rate limits
5. **Isolate from production** – Never mix test and production data
