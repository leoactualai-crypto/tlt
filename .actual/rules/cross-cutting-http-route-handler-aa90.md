# Adopt Async Function Definitions for HTTP Route Handlers: Http Route Handler

These rules are ALWAYS ACTIVE for all HTTP route handler functions in FastAPI services (event_manager, rsvp, discord_adapter) and any other services exposing HTTP APIs through the FastAPI framework.

### Rules

- **R-HTTP-001** MUST: All HTTP route handler functions MUST be defined with `async def` syntax.

### Verify

```bash
# Check for synchronous route handlers decorated with @router or @app
grep -r '@router\.(get|post|put|delete|patch)' --include='*.py' -A 1 | grep -v 'async def' | grep 'def ' && echo 'FAIL: Sync handlers found' || echo 'PASS: All handlers async'

# Verify no sync route handlers exist in service code
grep -r 'def.*router\.' --include='*.py' | grep -v 'async def' && echo 'Found sync route handlers' || echo 'All route handlers are async'

# Run async-specific tests
python -m pytest tests/ -k 'test_async' -v --tb=short
```

**Accept when:**
- All route handler functions decorated with `@router.*` or `@app.*` use `async def` syntax
- Grep verification commands return no synchronous route handlers in service code
- Code review confirms no blocking I/O operations (time.sleep, synchronous database calls) within async route handlers
- All FastAPI route handlers in MCP services (event_manager, rsvp) are async
- All FastAPI route handlers in adapter services (discord_adapter) are async
- Health check, ping, and monitoring endpoints use async def
- CRUD operations for RSVP, experience, and event management use async def

<enforcement>
Claude Code MUST NOT skip or defer verification. All route handlers must be verified as async before accepting changes to HTTP endpoint definitions.
</enforcement>