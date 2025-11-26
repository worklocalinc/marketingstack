# Stack Coordinator

You are coordinating changes across the WorkLocal Marketing Stack.

## Context
Read these files first:
- `docs/repos-and-domains.md` - All services, repos, and domains
- `docs/architecture.md` - Service boundaries and data flow
- `docs/contracts-ids-and-urls.md` - ID contracts
- `docs/contracts-events.md` - Event schemas

## Services
| Service | Repo | Domain |
|---------|------|--------|
| Affiliate Network | `worklocalinc/AffiliateTracker` | `adjump.com` |
| Domain Steward | `worklocalinc/domains.worklocal.ai` | `domains.adjump.com` |
| Messaging Core | `work-local-inc/messigingcore` | `messages.adjump.com` |
| Creative Generator | `work-local-inc/creatives` | `creativebuilder.adjump.com` |
| Offer Creator | `work-local-inc/superaffiliatesystem` | `houseoffers.adjump.com` |
| Email Seeder | `work-local-inc/seed` | `seed.adjump.com` |

## Task
Analyze the user's request: $ARGUMENTS

1. Identify which services are affected
2. Check if any ID contracts need to be preserved
3. Determine the order of changes (which service first?)
4. List files that need modification in each service
5. Warn about any contract violations

Output a clear plan with:
- Services affected
- Changes needed per service
- Contract considerations
- Recommended implementation order
