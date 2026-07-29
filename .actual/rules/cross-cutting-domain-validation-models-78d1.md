# Adopt Pydantic BaseModel for Domain Validation in API Boundaries: Domain Validation Models

These rules are ALWAYS ACTIVE for all FastAPI router endpoint request/response models, domain entities exposed through HTTP APIs, data transfer objects crossing service boundaries, configuration models requiring validation, and monitoring/health check response structures.

### Rules

- **R-DOMAIN-001** MUST: Domain validation models MUST use Pydantic BaseModel as the base class for all API boundary request/response models.
- **R-DOMAIN-002** MUST: Domain validation models MUST use Pydantic Field constraints (ge, le, description) for numeric bounds and field documentation.
- **R-DOMAIN-003** MUST: FastAPI router decorators MUST specify response_model parameter using Pydantic models for automatic validation and serialization.
- **R-DOMAIN-004** SHOULD: Complex nested models SHOULD use Pydantic's update_forward_refs() for circular dependencies and prefer flat models at API boundaries.
- **R-DOMAIN-005** MAY: Performance-critical paths MAY request exception (EXC-002) if demonstrating measurable Pydantic overhead (>10ms p99 latency) with benchmark data.

### Verify

```bash
# Count BaseModel subclasses in adapter and service modules
grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | grep -v '__pycache__' | wc -l

# Count response_model usage in FastAPI routes
grep -r 'response_model=' --include='*.py' monorepo/tlt/adapters/ monorepo/tlt/services/ | wc -l

# Verify Pydantic import works
python -c 'from pydantic import BaseModel, Field; m = BaseModel(); print("Pydantic import successful")'
```

**Accept when:**
- All API endpoint files contain at least one BaseModel subclass for request or response validation
- FastAPI router decorators specify response_model parameter using Pydantic models
- Grep commands show consistent pattern of BaseModel usage across adapter and service modules
- Malformed requests to endpoints return 422 Unprocessable Entity with detailed Pydantic validation error messages

<enforcement>
Claude Code MUST NOT skip or defer verification. All new API endpoints MUST include Pydantic BaseModel validation models with Field constraints before merge approval. Violations trigger PR comments requesting model addition or CI pipeline failures on detection of unvalidated dictionary parameters in FastAPI routes.
</enforcement>