# Standardize FastAPI Router Decorators for Internal Service Endpoint Definitions: Services Use Loguru

These rules are ALWAYS ACTIVE for all Python-based internal services within the monorepo that expose HTTP endpoints via FastAPI routers, including MCP services (event_manager, rsvp) and adapter services (discord_adapter).

### Rules

- **R-FASTAPI-001** MUST: All services MUST use loguru logger for consistent logging across endpoint handlers.

### Verify

```bash
# Verify all router handlers are async
grep -r '@router\.(get|post|put|delete)' monorepo/tlt/mcp_services/ monorepo/tlt/adapters/ | grep -v 'async def' && echo 'FAIL: Found non-async handlers' || echo 'PASS: All handlers are async'

# Verify health endpoints use router.get decorator
grep -r 'def health_check\|def ping' monorepo/tlt/mcp_services/ monorepo/tlt/adapters/ | grep -v '@router\.get' && echo 'FAIL: Health endpoints not using router.get' || echo 'PASS: Health endpoints properly decorated'

# Verify all services import and use loguru
python -c "import ast; import sys; files = ['monorepo/tlt/mcp_services/event_manager/main.py', 'monorepo/tlt/mcp_services/rsvp/main.py']; [sys.exit(1) for f in files if 'from loguru import logger' not in open(f).read()]; print('PASS: All services use loguru')"
```

**Accept when:**
- All service files in mcp_services/ and adapters/ directories use @router decorator patterns with async function definitions
- Every service implements /health endpoint returning JSON with status field and service-specific metrics
- All verification commands pass without detecting synchronous handlers or missing health endpoints
- All services import loguru logger and use it for endpoint handler logging

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verification commands must pass before accepting changes to services in scope.
</enforcement>