# Standardize FastAPI Router Decorators for Internal Service Endpoint Definitions: Internal Services Exposing

These rules are ALWAYS ACTIVE for all Python-based internal services within the monorepo that expose HTTP endpoints using FastAPI, including MCP services (event_manager, rsvp) and adapter services (discord_adapter).

### Rules

- **R-FASTAPI-001** MUST: All internal services exposing HTTP endpoints MUST use FastAPI APIRouter instances with explicit decorator patterns (@router.get, @router.post, @router.put, @router.delete).
- **R-FASTAPI-002** MUST: All router-decorated endpoint handlers MUST be defined as async functions.
- **R-FASTAPI-003** MUST: Every service MUST implement a /health endpoint returning JSON with a status field and service-specific metrics.
- **R-FASTAPI-004** MUST: All services MUST use loguru for logging (from loguru import logger).
- **R-FASTAPI-005** SHOULD: Endpoint handlers SHOULD include response_model specifications using Pydantic models for type safety.

### Verify

```bash
# Check for non-async handlers in router decorators
grep -r '@router\.(get|post|put|delete)' monorepo/tlt/mcp_services/ monorepo/tlt/adapters/ | grep -v 'async def' && echo 'FAIL: Found non-async handlers' || echo 'PASS: All handlers are async'

# Check that health endpoints use router.get decorator
grep -r 'def health_check\|def ping' monorepo/tlt/mcp_services/ monorepo/tlt/adapters/ | grep -v '@router\.get' && echo 'FAIL: Health endpoints not using router.get' || echo 'PASS: Health endpoints properly decorated'

# Verify all services import loguru
python -c "import ast; import sys; files = ['monorepo/tlt/mcp_services/event_manager/main.py', 'monorepo/tlt/mcp_services/rsvp/main.py']; [sys.exit(1) for f in files if 'from loguru import logger' not in open(f).read()]; print('PASS: All services use loguru')"
```

**Accept when:**
- All service files in mcp_services/ and adapters/ directories use @router decorator patterns with async function definitions
- Every service implements /health endpoint returning JSON with status field and service-specific metrics
- All grep verification commands pass without detecting synchronous handlers or missing health endpoints
- All services import loguru for logging
- Response models are specified for endpoint handlers using Pydantic models

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for internal service implementations.
</enforcement>