# Standardize Pydantic BaseModel for Internal API Request/Response Validation: Basemodel Classes Include

These rules are ALWAYS ACTIVE for all FastAPI router endpoints in Discord adapters (reminder.py, rsvp.py, event.py), TLT services (event_manager.py), and structured data contracts in MCP services (photo_processor.py) that handle request/response validation.

### Rules

- **R-PYDANTIC-001** SHOULD: BaseModel classes SHOULD include Field description parameters to document field semantics for API consumers and generated documentation.

### Verify

```bash
# Count BaseModel class definitions across scope
grep -r 'class.*BaseModel' monorepo/tlt/adapters monorepo/tlt/services monorepo/tlt/mcp_services --include='*.py' | wc -l

# Count router endpoints with response_model declarations
grep -r '@router\.(post|put|patch).*response_model=' monorepo/tlt/adapters monorepo/tlt/services --include='*.py' | wc -l

# Count Field descriptors with constraints or descriptions
grep -r 'Field(' monorepo/tlt/adapters monorepo/tlt/services monorepo/tlt/mcp_services --include='*.py' | grep -E '(ge=|le=|min_length=|max_length=|description=)' | wc -l
```

**Accept when:**
- All FastAPI router endpoints accepting structured request bodies define corresponding BaseModel subclasses with explicit field types
- All FastAPI router endpoints returning structured responses declare response_model parameter with BaseModel subclass
- BaseModel classes with numeric or string fields include Field descriptors with appropriate constraints (ge, le, min_length, etc.) where domain invariants exist
- New BaseModel classes include Field(description='...') for all fields to enable self-documenting OpenAPI schemas

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All new endpoints and schema modifications must be validated against the verify commands before approval.
</enforcement>