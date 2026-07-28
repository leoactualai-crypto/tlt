# Adopt Async Function Definitions for HTTP Route Handlers: Services Use Synchronous

These rules are ALWAYS ACTIVE for all FastAPI route handlers in MCP services (event_manager, rsvp) and adapter services (discord_adapter), including health check, ping, monitoring endpoints, and CRUD operations for RSVP, experience, and event management.

### Rules

- **R-ASYNC-001** MUST: All FastAPI route handlers decorated with `@router.*` or `@app.*` use `async def` syntax, except when performing pure computation with no I/O and blocking behavior is explicitly acceptable.
- **R-ASYNC-002** MUST: Route handlers performing any I/O operations (database access, HTTP calls, external API integration) must use `async def` to prevent blocking the event loop.
- **R-ASYNC-003** SHOULD: Health check, ping, and monitoring endpoints use `async def` even when performing minimal I/O to maintain consistency across the codebase.
- **R-ASYNC-004** SHOULD: Redirect handlers and URL rewriting endpoints use `async def` with immediate return of RedirectResponse to follow the established pattern.
- **R-ASYNC-005** MAY: Services MAY use synchronous route handlers only when the handler performs pure computation with no I/O and blocking behavior is explicitly acceptable (EXC-001).

### Verify

```bash
# Check for any synchronous route handlers in service code
grep -r 'def.*router\.' --include='*.py' monorepo/tlt/ | grep -v 'async def' && echo 'Found sync route handlers' || echo 'All route handlers are async'

# Verify all @router.* decorated functions use async def
grep -r '@router\.(get|post|put|delete|patch)' --include='*.py' -A 1 monorepo/tlt/ | grep -v 'async def' | grep 'def ' && echo 'FAIL: Sync handlers found' || echo 'PASS: All handlers async'

# Run async-specific tests
python -m pytest tests/ -k 'test_async' -v --tb=short
```

**Accept when:**
- All route handler functions decorated with `@router.*` or `@app.*` use `async def` syntax
- Grep verification commands return no synchronous route handlers in service code
- Code review confirms no blocking I/O operations (time.sleep, synchronous database calls) within async route handlers
- All FastAPI route handlers in event_manager, rsvp, and discord_adapter services are asynchronous

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks, CI pipeline static analysis, code review checklists, and automated tests MUST validate async/await usage. Violations cause CI build failure and code review blocks until remediated or documented exception is approved.
</enforcement>