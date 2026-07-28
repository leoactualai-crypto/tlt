# Adopt Async Function Definitions for HTTP Route Handlers: Health Check Endpoints

These rules are ALWAYS ACTIVE for all FastAPI route handlers in MCP services (event_manager, rsvp) and adapter services (discord_adapter), including health check, ping, monitoring endpoints, and CRUD operations for RSVP, experience, and event management.

### Rules

- **R-ASYNC-001** MUST: Health check endpoints MUST be implemented as async functions to maintain consistency with the service concurrency model.
- **R-ASYNC-002** MUST: All FastAPI route handlers decorated with `@router.*` or `@app.*` MUST use `async def` syntax.
- **R-ASYNC-003** MUST: Route handlers MUST NOT contain blocking I/O operations (e.g., `time.sleep`, synchronous database calls) without explicit executor delegation.
- **R-ASYNC-004** SHOULD: Use async-compatible libraries (asyncpg, aiohttp, loguru) for all I/O operations within async route handlers.
- **R-ASYNC-005** SHOULD: Implement health check endpoints with `async def` even when they perform minimal I/O to maintain consistency.

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
- All route handler functions decorated with `@router.*` or `@app.*` use `async def` syntax
- Grep verification commands return no synchronous route handlers in service code
- Code review confirms no blocking I/O operations (time.sleep, synchronous database calls) within async route handlers
- All health check, ping, and monitoring endpoints are implemented as async functions

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks, CI pipeline static analysis, code review checklists, and automated tests MUST validate async/await usage. Violations result in CI build failure and code review blocking until remediated or documented exception is approved.
</enforcement>