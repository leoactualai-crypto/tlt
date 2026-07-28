# Adopt Async Function Definitions for HTTP Route Handlers: Redirect Handlers Simple

These rules are ALWAYS ACTIVE for all FastAPI route handlers in MCP services (event_manager, rsvp), adapter services (discord_adapter), health check endpoints, CRUD operations, and redirect handlers across the monorepo.

### Rules

- **R-ASYNC-001** SHOULD: Redirect handlers and simple response endpoints SHOULD follow the async pattern even when not performing I/O to maintain architectural consistency.
- **R-ASYNC-002** MUST: All FastAPI route handlers decorated with `@router.*` or `@app.*` MUST use `async def` syntax.
- **R-ASYNC-003** MUST: Async route handlers MUST NOT contain blocking I/O operations (e.g., `time.sleep`, synchronous database calls, synchronous file I/O).
- **R-ASYNC-004** SHOULD: Health check, ping, and monitoring endpoints SHOULD be implemented with `async def` to maintain consistency.
- **R-ASYNC-005** MAY: Unavoidable blocking operations within async handlers MAY use `run_in_executor` for delegation to thread pools.

### Verify

```bash
# Check for synchronous route handlers
grep -r 'def.*router\.' --include='*.py' monorepo/tlt/ | grep -v 'async def' && echo 'Found sync route handlers' || echo 'All route handlers are async'

# Verify all decorated handlers are async
grep -r '@router\.(get|post|put|delete|patch)' --include='*.py' -A 1 monorepo/tlt/ | grep -v 'async def' | grep 'def ' && echo 'FAIL: Sync handlers found' || echo 'PASS: All handlers async'

# Run async-specific tests
python -m pytest tests/ -k 'test_async' -v --tb=short
```

**Accept when:**
- All route handler functions decorated with `@router.*` or `@app.*` use `async def` syntax
- Grep verification commands return no synchronous route handlers in service code
- Code review confirms no blocking I/O operations (`time.sleep`, synchronous database calls) within async route handlers
- All health check, redirect, and simple response endpoints are implemented with `async def`

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks, CI pipeline static analysis, code review checklists, and automated tests MUST validate async/await usage. Violations result in CI build failure and code review blocking until remediated or documented exception is approved.
</enforcement>