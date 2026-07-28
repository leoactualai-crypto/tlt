# Adopt Async Function Definitions for HTTP Route Handlers: Route Handlers Performing

These rules are ALWAYS ACTIVE for all FastAPI route handlers in MCP services (event_manager, rsvp) and adapter services (discord_adapter), including health check, ping, monitoring endpoints, and CRUD operations for RSVP, experience, and event management.

### Rules

- **R-ASYNC-001** MUST: Route handlers performing I/O operations (database queries, external API calls, file operations) MUST use await for asynchronous execution.

### Verify

```bash
# Check for synchronous route handlers
grep -r 'def.*router\.' --include='*.py' monorepo/tlt/ | grep -v 'async def' && echo 'Found sync route handlers' || echo 'All route handlers are async'

# Verify all decorated handlers are async
grep -r '@router\.\(get\|post\|put\|delete\|patch\)' --include='*.py' -A 1 monorepo/tlt/ | grep -v 'async def' | grep 'def ' && echo 'FAIL: Sync handlers found' || echo 'PASS: All handlers async'

# Run async-specific tests
python -m pytest tests/ -k 'test_async' -v --tb=short
```

**Accept when:**
- All route handler functions decorated with @router.* or @app.* use async def syntax
- Grep verification commands return no synchronous route handlers in service code
- Code review confirms no blocking I/O operations (time.sleep, synchronous database calls) within async route handlers

<enforcement>
Claude Code MUST NOT skip or defer verification. All route handlers must be verified as async before acceptance.
</enforcement>