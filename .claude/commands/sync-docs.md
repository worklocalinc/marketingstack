# Sync Documentation

Keep documentation synchronized across the marketing stack.

## Documentation Locations
- `README.md` - Main overview, service table
- `docs/repos-and-domains.md` - Central reference for repos, domains, envs
- `docs/architecture.md` - Service boundaries, data flow
- `docs/contracts-ids-and-urls.md` - ID contracts, URL format
- `docs/contracts-events.md` - Event schemas
- `ai/README.md` - AI agent guidance
- `services/*/README.md` - Per-service documentation

## Task
Sync docs for: $ARGUMENTS

Check for inconsistencies:
1. **Service table** - Is the service listed in all relevant docs?
2. **Domains** - Are domains consistent across all files?
3. **IDs** - Are canonical IDs documented everywhere?
4. **Endpoints** - Do service READMEs match actual endpoints?
5. **Environment variables** - Are they documented in repos-and-domains.md?

When adding a new service, update:
1. `README.md` - Add to service table
2. `docs/repos-and-domains.md` - Full entry with domain, env vars
3. `docs/architecture.md` - Service boundaries section
4. `ai/README.md` - Add to principles and navigation
5. Create `services/<name>/README.md`
6. Create `docs/<name>.md` if needed

Output a list of files that need updates and the specific changes required.
