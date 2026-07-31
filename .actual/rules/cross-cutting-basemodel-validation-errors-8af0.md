# Standardize Pydantic BaseModel for Internal API Request/Response Validation: Basemodel Validation Errors

These rules are ALWAYS ACTIVE for all FastAPI router endpoints within the monorepo that handle structured request/response data, including Discord adapters (reminder.py, rsvp.py, event.py), TLT services (event_manager.py), and MCP services (photo_processor.py).

### Rules

- **R-PYDANTIC-001** SHOULD: BaseModel validation errors SHOULD propagate as HTTP 422 responses with detailed field-level error messages via FastAPI default exception handling.
- **R-PYDANTIC-002** MUST: All FastAPI router endpoints accepting structured request bodies MUST define corresponding BaseModel subclasses with explicit field types.
- **R-PYDANTIC-003** MUST: All FastAPI router endpoints returning structured responses MUST declare response_model parameter with BaseModel subclass.
- **R-PYDANTIC-004** SHOULD: BaseModel classes with numeric or string fields SHOULD include Field descriptors with appropriate constraints (ge, le, min_length, max_length, description) where domain invariants exist.
- **R-PYDANTIC-005** SHOULD: BaseModel classes SHOULD be defined in dedicated schema modules (e.g., schemas.py) separate from router definitions to enable reuse across endpoints and testing.
- **R-PYDANTIC-006** SHOULD: Pydantic Field descriptors SHOULD include description parameters to generate self-documenting OpenAPI schemas.
- **R-PYDANTIC-007** MAY: Complex validation logic MAY use Pydantic @validator decorators with pre=True or post=True to control validation order and access to other fields.

### Verify

```bash
# Count BaseModel class definitions across scope
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
- No new router endpoints are detected without corresponding BaseModel definitions.
- No endpoints accept Dict[str, Any] without documented exception approval.

<enforcement>
Claude Code MUST NOT skip or defer verification. All new or modified FastAPI endpoints MUST be checked against these rules before approval. Code review MUST verify BaseModel definitions and response_model declarations. CI pipeline MUST run verification commands to enforce compliance.
</enforcement>