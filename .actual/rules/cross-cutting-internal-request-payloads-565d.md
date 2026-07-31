# Standardize Pydantic BaseModel for Internal API Request/Response Validation: Internal Request Payloads

These rules are ALWAYS ACTIVE for all FastAPI router endpoints in Discord adapters, TLT services, and MCP services that handle structured request/response data within the monorepo.

### Rules

- **R-PYDANTIC-001** MUST: All internal API request payloads MUST be defined as Pydantic BaseModel subclasses with explicit field type annotations.
- **R-PYDANTIC-002** MUST: All FastAPI router endpoints returning structured responses MUST declare response_model parameter with BaseModel subclass.
- **R-PYDANTIC-003** MUST: BaseModel classes with numeric or string fields MUST include Field descriptors with appropriate constraints (ge, le, min_length, max_length, description) where domain invariants exist.
- **R-PYDANTIC-004** SHOULD: Define BaseModel classes in dedicated schema modules (e.g., schemas.py) separate from router definitions to enable reuse across endpoints and testing.
- **R-PYDANTIC-005** SHOULD: Use Pydantic Field descriptors with description parameters to generate self-documenting OpenAPI schemas.
- **R-PYDANTIC-006** MAY: Legacy endpoints undergoing migration may temporarily accept Dict[str, Any] with explicit validation logic until BaseModel migration is complete (EXC-001).
- **R-PYDANTIC-007** MAY: Proxy endpoints forwarding opaque payloads to downstream services may skip BaseModel validation if payload structure is not interpreted (EXC-002).

### Verify

```bash
# Count BaseModel definitions across adapter and service files
grep -r 'class.*BaseModel' monorepo/tlt/adapters monorepo/tlt/services monorepo/tlt/mcp_services --include='*.py' | wc -l

# Count response_model declarations in router endpoints
grep -r '@router\.(post|put|patch).*response_model=' monorepo/tlt/adapters monorepo/tlt/services --include='*.py' | wc -l

# Count Field descriptors with constraints
grep -r 'Field(' monorepo/tlt/adapters monorepo/tlt/services monorepo/tlt/mcp_services --include='*.py' | grep -E '(ge=|le=|min_length=|max_length=|description=)' | wc -l
```

**Accept when:**
- All FastAPI router endpoints accepting structured request bodies define corresponding BaseModel subclasses with explicit field types.
- All FastAPI router endpoints returning structured responses declare response_model parameter with BaseModel subclass.
- BaseModel classes with numeric or string fields include Field descriptors with appropriate constraints (ge, le, min_length, etc.) where domain invariants exist.
- New endpoints do not accept Dict[str, Any] without documented exception approval (EXC-001 or EXC-002).

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All new internal API endpoints MUST comply with R-PYDANTIC-001, R-PYDANTIC-002, and R-PYDANTIC-003 unless an explicit exception (EXC-001 or EXC-002) is documented in the endpoint implementation.
</enforcement>