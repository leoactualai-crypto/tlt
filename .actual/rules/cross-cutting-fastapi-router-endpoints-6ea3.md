# Standardize Pydantic BaseModel for Internal API Request/Response Validation: Fastapi Router Endpoints

These rules are ALWAYS ACTIVE for all FastAPI router endpoints in Discord adapters, TLT services, and MCP services that handle structured request/response data within the monorepo.

### Rules

- **R-PYDANTIC-001** MUST: FastAPI router endpoints MUST NOT accept untyped `Dict` or `Any` parameters for structured request bodies.
- **R-PYDANTIC-002** MUST: All FastAPI router endpoints accepting structured request bodies MUST define corresponding `BaseModel` subclasses with explicit field types.
- **R-PYDANTIC-003** MUST: All FastAPI router endpoints returning structured responses MUST declare `response_model` parameter with `BaseModel` subclass in router decorator.
- **R-PYDANTIC-004** SHOULD: `BaseModel` classes with numeric or string fields SHOULD include `Field` descriptors with appropriate constraints (`ge`, `le`, `min_length`, `max_length`, `description`) where domain invariants exist.
- **R-PYDANTIC-005** SHOULD: `BaseModel` classes SHOULD be defined in dedicated schema modules (e.g., `schemas.py`) separate from router definitions to enable reuse across endpoints and testing.
- **R-PYDANTIC-006** MAY: Legacy endpoints undergoing migration MAY temporarily accept `Dict[str, Any]` with explicit validation logic until `BaseModel` migration is complete (EXC-001).
- **R-PYDANTIC-007** MAY: Proxy endpoints forwarding opaque payloads to downstream services MAY skip `BaseModel` validation if payload structure is not interpreted (EXC-002).

### Verify

```bash
# Count BaseModel class definitions across adapter and service files
grep -r 'class.*BaseModel' monorepo/tlt/adapters monorepo/tlt/services monorepo/tlt/mcp_services --include='*.py' | wc -l

# Count router endpoints with response_model declarations
grep -r '@router\.(post|put|patch).*response_model=' monorepo/tlt/adapters monorepo/tlt/services --include='*.py' | wc -l

# Count Field descriptors with constraints (ge, le, min_length, max_length, description)
grep -r 'Field(' monorepo/tlt/adapters monorepo/tlt/services monorepo/tlt/mcp_services --include='*.py' | grep -E '(ge=|le=|min_length=|max_length=|description=)' | wc -l
```

**Accept when:**
- All FastAPI router endpoints accepting structured request bodies define corresponding `BaseModel` subclasses with explicit field types.
- All FastAPI router endpoints returning structured responses declare `response_model` parameter with `BaseModel` subclass.
- `BaseModel` classes with numeric or string fields include `Field` descriptors with appropriate constraints (`ge`, `le`, `min_length`, etc.) where domain invariants exist.
- No new router endpoints are detected accepting `Dict[str, Any]` without documented exception approval (EXC-001 or EXC-002).
- Static type analysis with mypy confirms type annotations on router endpoint parameters and return types.

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. CI pipeline MUST run grep-based verification commands to count BaseModel usage and response_model declarations. Code review MUST verify new endpoints define BaseModel request/response schemas. Static analysis with mypy MUST enforce type annotations. Violations block merge unless documented exception is approved by tech lead.
</enforcement>