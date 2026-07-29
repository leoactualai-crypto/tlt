# Adopt Pydantic BaseModel for Domain Validation in API Boundaries: Endpoints Not Accept

These rules are ALWAYS ACTIVE for all FastAPI-based service adapters, HTTP endpoints, domain model definitions, and API boundary implementations within the codebase.

### Rules

- **R-PYDANTIC-001** MUST_NOT: API endpoints MUST NOT accept unvalidated dictionaries or raw JSON without Pydantic model validation.

### Verify

```bash
# Count BaseModel subclasses across adapter and service modules
grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/ | grep -v '__pycache__' | wc -l

# Count response_model declarations in FastAPI routes
grep -r 'response_model=' --include='*.py' monorepo/tlt/adapters/ monorepo/tlt/services/ | wc -l

# Verify Pydantic import works
python -c 'from pydantic import BaseModel, Field; m = BaseModel(); print("Pydantic import successful")'
```

**Accept when:**
- All API endpoint files contain at least one BaseModel subclass for request or response validation
- FastAPI router decorators specify response_model parameter using Pydantic models
- Grep commands show consistent pattern of BaseModel usage across adapter and service modules
- No FastAPI routes accept unvalidated dictionary or raw JSON parameters

<enforcement>
Clause R-PYDANTIC-001 is mandatory. Code review MUST verify all new API endpoints use Pydantic BaseModel for request/response validation. CI pipeline MUST detect and fail on unvalidated dictionary parameters in FastAPI routes. Violations require Pydantic model addition before merge approval.
</enforcement>