# Domain Manager

Manage domains across the WorkLocal Marketing Stack.

## Domain Inventory
| Domain | Service | Purpose |
|--------|---------|---------|
| `adjump.com` | Affiliate Network | Click tracking |
| `domains.adjump.com` | Domain Steward | Domain management API |
| `messages.adjump.com` | Messaging Core | Messaging API |
| `creativebuilder.adjump.com` | Creative Generator | Creative API |
| `houseoffers.adjump.com` | Offer Creator | Offer API |
| `seed.adjump.com` | Email Seeder | Test email API |

## Domain Roles (managed by Domain Steward)
- **tracking** - Click tracking domains (e.g., `adjump.com/click`)
- **lander** - Offer landing pages
- **email** - Email sending domains
- **cdn** - Asset delivery

## Task
Domain operation: $ARGUMENTS

For domain operations:
1. Check if domain exists in Domain Steward inventory
2. Verify DNS configuration requirements
3. Check domain role assignments
4. Validate SSL/Cloudflare setup

For adding new domains:
1. Add to Domain Steward via API
2. Configure DNS records
3. Assign domain role
4. Update `docs/repos-and-domains.md`

Output the required API calls or configuration steps.
