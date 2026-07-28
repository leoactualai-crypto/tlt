# Adopt Async Function Definitions for HTTP Route Handlers: Services Use Fastapi

These rules are ALWAYS ACTIVE for all FastAPI route handlers in MCP services (event_manager, rsvp) and adapter services (discord_adapter), including health check, ping, monitoring endpoints, and CRUD operations for RSVP, experience, and event management.

### Rules

- **R-ASYNC-001** SHOULD: Services SHOULD use FastAPI framework decorators (@router.get, @router.post, etc.) with async route handlers

### Verify

```bash
# Check for any synchronous route handlers
grep -r 'def.*router\.' --include='*.py' monorepo/tlt/ | grep -v 'async def' && echo 'Found sync route handlers' || echo 'All route handlers are async'

# Verify all @router.* decorators use async def
grep -r '@router\.(get|post|put|delete|patch)' --include='*.py' -A 1 monorepo/tlt/ | grep -v 'async def' | grep 'def ' && echo 'FAIL: Sync handlers found' || echo 'PASS: All handlers async'

# Run async-specific tests
python -m pytest tests/ -k 'test_async' -v --tb=short
```

**Accept when:**
- All route handler functions decorated with @router.* or @app.* use async def syntax
- Grep verification commands return no synchronous route handlers in service code
- Code review confirms no blocking I/O operations (time.sleep, synchronous database calls) within async route handlers

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks, CI pipeline static analysis, code review checklists, and automated tests MUST validate async/await usage in all route handlers. Violations block CI builds and code review merges until remediated or documented exceptions are approved.
</enforcement>