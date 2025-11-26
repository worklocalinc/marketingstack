# Email Seeder – Architecture Overview

The **Email Seeder** service provides pre-configured test email accounts for QA workflows across the WorkLocal marketing stack.

## Role in the Stack

```
                    ┌─────────────────┐
                    │  Email Seeder   │
                    │  (Test Emails)  │
                    └────────┬────────┘
                             │
           ┌─────────────────┼─────────────────┐
           │                 │                 │
           ▼                 ▼                 ▼
┌──────────────────┐ ┌──────────────┐ ┌──────────────────┐
│  Messaging Core  │ │Offer Creator │ │ Affiliate Network│
│  (Sends to test  │ │ (QA offers)  │ │ (Track test      │
│   addresses)     │ │              │ │  conversions)    │
└──────────────────┘ └──────────────┘ └──────────────────┘
```

## Responsibilities

### Owns
- `test_accounts` – Managed mailbox configurations
- `test_emails` – Pre-configured email addresses
- `inbox_messages` – Captured incoming emails
- `qa_test_runs` – Automated test executions

### Does NOT Own
- Production contacts (Messaging Core)
- Email sending (Messaging Core)
- Offer definitions (Offer Creator)
- Click/conversion tracking (Affiliate Network)
- Domain configuration (Domain Steward)

## Core IDs

| ID | Table | Purpose |
|----|-------|---------|
| `testAccountId` | `test_accounts.id` | Managed mailbox identity |
| `testEmailId` | `test_emails.id` | Specific test email identity |
| `qaTestRunId` | `qa_test_runs.id` | Test execution identity |

## Integration Points

### Email Seeder → Messaging Core

When running QA tests, Email Seeder registers test emails as contacts:

```http
POST https://msg.worklocal.dev/api/contacts
Content-Type: application/json

{
  "email": "test-signup-001@seeder.worklocal.dev",
  "source": "email_seeder",
  "metadata": {
    "testEmailId": "te_abc123",
    "testPurpose": "offer_validation"
  }
}
```

### Email Seeder → Affiliate Network

After test flows complete, Email Seeder queries for conversion verification:

```http
GET https://trk.worklocal.dev/api/conversions?sub1=<contactId>
```

Response validates that the test generated expected tracking data.

### Email Seeder ← Other Services

Other services can request available test emails:

```http
GET https://seeder.worklocal.dev/api/test-emails?purpose=signup&vertical=jobs
```

Returns:
```json
{
  "testEmail": {
    "id": "te_abc123",
    "email": "jobs-test-042@seeder.worklocal.dev",
    "purpose": "signup",
    "verticalTags": ["jobs", "employment"]
  }
}
```

## QA Workflow Automation

### Signup Test Flow

```
1. GET /api/test-emails?purpose=signup
   → Receive available test email

2. Submit signup form on offer lander
   → Contact created in Messaging Core

3. Wait for welcome email
   → Email Seeder polls inbox

4. Extract and click tracking URL
   → Click recorded in Affiliate Network

5. Complete conversion action
   → Conversion recorded in Affiliate Network

6. POST /api/qa/results
   → Record pass/fail with full trace
```

### Test Result Schema

```json
{
  "qaTestRunId": "qtr_xyz789",
  "testType": "signup",
  "testEmailId": "te_abc123",
  "status": "passed",
  "timeline": [
    {"step": "email_acquired", "at": "2024-01-15T10:00:00Z"},
    {"step": "signup_submitted", "at": "2024-01-15T10:00:05Z"},
    {"step": "email_received", "at": "2024-01-15T10:00:32Z"},
    {"step": "link_clicked", "at": "2024-01-15T10:00:35Z"},
    {"step": "conversion_recorded", "at": "2024-01-15T10:00:40Z"}
  ],
  "artifacts": {
    "contactId": "c_def456",
    "messageId": "m_ghi789",
    "clickId": "clk_jkl012",
    "conversionId": "conv_mno345"
  }
}
```

## Email Provider Support

| Provider | Use Case | Notes |
|----------|----------|-------|
| Gmail (API) | High-fidelity testing | OAuth required |
| IMAP Generic | Bulk test accounts | Any IMAP provider |
| Mailinator | Disposable testing | Public inboxes |
| Custom SMTP | Self-hosted | Full control |

## Security Considerations

1. **Credential Encryption** – All email credentials stored encrypted
2. **Test Isolation** – Test emails clearly tagged, never mixed with production
3. **Rate Limiting** – Respect provider API limits
4. **Data Retention** – Auto-purge old inbox messages and test runs
5. **Access Control** – API key required for all endpoints

## Environment Configuration

```bash
# Core
DATABASE_URL=postgres://...
API_KEY=esk_...
ENCRYPTION_KEY=...

# Gmail OAuth (optional)
GMAIL_CLIENT_ID=...
GMAIL_CLIENT_SECRET=...
GMAIL_REDIRECT_URI=...

# Generic IMAP
IMAP_DEFAULT_PORT=993
IMAP_DEFAULT_TLS=true

# Service Integration
MESSAGING_CORE_URL=https://msg.worklocal.dev
AFFILIATE_NETWORK_URL=https://trk.worklocal.dev
OFFER_CREATOR_URL=https://offers.worklocal.dev
```

## Recommended Test Email Naming

Use consistent naming for easy identification:

```
{vertical}-{purpose}-{number}@seeder.worklocal.dev

Examples:
  jobs-signup-001@seeder.worklocal.dev
  finance-offer-042@seeder.worklocal.dev
  health-automation-007@seeder.worklocal.dev
```

## Future Enhancements

- SMS test numbers for SMS channel testing
- Push notification test endpoints
- Scheduled QA runs with alerting
- Integration with CI/CD pipelines
- Visual email rendering comparison
