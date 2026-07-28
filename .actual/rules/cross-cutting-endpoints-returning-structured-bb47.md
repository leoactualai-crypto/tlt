# Standardize FastAPI Router Decorators for Internal Service Endpoint Definitions: Endpoints Returning Structured

These rules are ALWAYS ACTIVE for all Python-based internal services within the monorepo that expose HTTP endpoints using FastAPI, including MCP services (event_manager, rsvp) and adapter services (discord_adapter).

### Rules

- **R-FASTAPI-001** SHOULD: Endpoints returning structured data SHOULD specify `response_model` parameter in router decorators using Pydantic BaseModel classes.

### Verify

```bash
# Verify all handlers are async
grep -r '@router\.(get|post|put|delete)' monorepo/tlt/mcp_services/ monorepo/tlt/adapters/ | grep -v 'async def' && echo 'FAIL: Found non-async handlers' || echo 'PASS: All handlers are async'

# Verify health endpoints use router.get decorator
grep -r 'def health_check\|def ping' monorepo/tlt/mcp_services/ monorepo/tlt/adapters/ | grep -v '@router\.get' && echo 'FAIL: Health endpoints not using router.get' || echo 'PASS: Health endpoints properly decorated'

# Verify all services use loguru
python -c "import ast; import sys; files = ['monorepo/tlt/mcp_services/event_manager/main.py', 'monorepo/tlt/mcp_services/rsvp/main.py']; [sys.exit(1) for f in files if 'from loguru import logger' not in open(f).read()]; print('PASS: All services use loguru')"
```

**Accept when:**
- All service files in mcp_services/ and adapters/ directories use @router decorator patterns with async function definitions
- Every service implements /health endpoint returning JSON with status field and service-specific metrics
- All grep verification commands pass without detecting synchronous handlers or missing health endpoints
- Pydantic BaseModel classes are used for response_model specifications in router decorators

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks, CI pipeline integration tests, code review checklists, and automated OpenAPI schema validation are mandatory enforcement mechanisms. Violations block merge requests and trigger monitoring alerts.
</enforcement>