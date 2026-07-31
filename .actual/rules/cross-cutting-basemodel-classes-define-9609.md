# Standardize Pydantic BaseModel for Internal API Request/Response Validation: Basemodel Classes Define

These rules are ALWAYS ACTIVE for all FastAPI router endpoints in Discord adapters (reminder.py, rsvp.py, event.py), TLT services (event_manager.py), and structured data contracts in MCP services (photo_processor.py) that handle request/response validation.

### Rules

- **R-PYDANTIC-001** MUST: All FastAPI router endpoints accepting structured request bodies define corresponding BaseModel subclasses with explicit field types.
- **R-PYDANTIC-002** MUST: All FastAPI router endpoints returning structured responses declare response_model parameter with BaseModel subclass.
- **R-PYDANTIC-003** MUST: BaseModel classes with numeric or string fields include Field descriptors with appropriate constraints (ge, le, min_length, max_length, etc.) where domain invariants exist.
- **R-PYDANTIC-004** MAY: BaseModel classes MAY define custom validators using Pydantic @validator decorators for cross-field validation logic.
- **R-PYDANTIC-005** SHOULD: Define BaseModel classes in dedicated schema modules (e.g., schemas.py) separate from router definitions to enable reuse across endpoints and testing.
- **R-PYDANTIC-006** SHOULD: Use Pydantic Field descriptors with description parameters to generate self-documenting OpenAPI schemas.

### Verify

```bash
# Count BaseModel class definitions
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
Clause Code MUST NOT skip or defer verification. CI pipeline must run grep-based verification commands. Code review must verify new endpoints define BaseModel request/response schemas. Static analysis with mypy must enforce type annotations. Violations block merge unless exception is documented.
</enforcement>