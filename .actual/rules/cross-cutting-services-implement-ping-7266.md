# Standardize FastAPI Router Decorators for Internal Service Endpoint Definitions: Services Implement Ping

These rules are ALWAYS ACTIVE for all Python-based internal services within the monorepo that expose HTTP endpoints via FastAPI, including MCP services (event_manager, rsvp) and adapter services (discord_adapter).

### Rules

- **R-FASTAPI-PING-001** SHOULD: Services SHOULD implement a /ping endpoint for simple load balancer health checks returning minimal response payloads.

### Verify

```bash
# Verify all router handlers use async pattern
grep -r '@router\.(get|post|put|delete)' monorepo/tlt/mcp_services/ monorepo/tlt/adapters/ | grep -v 'async def' && echo 'FAIL: Found non-async handlers' || echo 'PASS: All handlers are async'

# Verify health endpoints are properly decorated
grep -r 'def health_check\|def ping' monorepo/tlt/mcp_services/ monorepo/tlt/adapters/ | grep -v '@router\.get' && echo 'FAIL: Health endpoints not using router.get' || echo 'PASS: Health endpoints properly decorated'

# Verify all services use loguru
python -c "import ast; import sys; files = ['monorepo/tlt/mcp_services/event_manager/main.py', 'monorepo/tlt/mcp_services/rsvp/main.py']; [sys.exit(1) for f in files if 'from loguru import logger' not in open(f).read()]; print('PASS: All services use loguru')"
```

**Accept when:**
- All service files in mcp_services/ and adapters/ directories use @router decorator patterns with async function definitions
- Every service implements /health endpoint returning JSON with status field and service-specific metrics
- All grep verification commands pass without detecting synchronous handlers or missing health endpoints
- Services use loguru for logging across all implementations

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks and CI pipeline integration tests are mandatory for all new service implementations and modifications to existing services.
</enforcement>