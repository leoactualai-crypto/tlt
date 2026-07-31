# Standardize Pydantic BaseModel for Internal API Request/Response Validation: Internal Response Contracts

These rules are ALWAYS ACTIVE for all FastAPI router endpoints within the monorepo that handle structured request/response data, including Discord adapters (reminder.py, rsvp.py, event.py), TLT services (event_manager.py), and MCP services (photo_processor.py).

### Rules

- **R-PYDANTIC-001** MUST: All internal API response contracts MUST be defined as Pydantic BaseModel subclasses and declared in FastAPI router response_model parameters.
- **R-PYDANTIC-002** MUST: All FastAPI router endpoints accepting structured request bodies MUST define corresponding BaseModel subclasses with explicit field types.
- **R-PYDANTIC-003** MUST: All FastAPI router endpoints returning structured responses MUST declare response_model parameter with BaseModel subclass.
- **R-PYDANTIC-004** SHOULD: BaseModel classes with numeric or string fields SHOULD include Field descriptors with appropriate constraints (ge, le, min_length, max_length, description) where domain invariants exist.
- **R-PYDANTIC-005** SHOULD: BaseModel classes SHOULD be defined in dedicated schema modules (e.g., schemas.py) separate from router definitions to enable reuse across endpoints and testing.
- **R-PYDANTIC-006** MAY: Legacy endpoints undergoing migration may temporarily accept Dict[str, Any] with explicit validation logic until BaseModel migration is complete (EXC-001).
- **R-PYDANTIC-007** MAY: Proxy endpoints forwarding opaque payloads to downstream services may skip BaseModel validation if payload structure is not interpreted (EXC-002).

### Verify

```bash
# Count BaseModel definitions across scope
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
- New router endpoints are not detected without corresponding BaseModel definitions.
- Endpoints do not accept Dict[str, Any] without documented exception approval.

<enforcement>
Claude Code MUST NOT skip or defer verification. CI pipeline verification commands MUST be executed. Code review checklist MUST require verification of new endpoints. Static analysis with mypy MUST enforce type annotations. Violations MUST block merge unless exception is approved and documented.
</enforcement>