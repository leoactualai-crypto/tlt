# Enforce Pydantic BaseModel for API Contract Validation in Public Interfaces: Validation Models Define

These rules are ALWAYS ACTIVE for all public API contracts, service boundaries, and domain validation models across the codebase, including MCP services (gateway, rsvp, event_manager, photo_vibe_check, guild_manager), agents (ambient_event_agent), Discord adapters, authentication contexts, and external API request/response contracts.

### Rules

- **R-VAL-001** MUST: Validation models MUST define explicit field types using typing annotations (str, int, float, datetime, Optional, List, Dict) with no implicit Any types for security-critical fields.
- **R-VAL-002** MUST: All public API request/response models in mcp_services, agents, and adapters MUST inherit from Pydantic BaseModel.
- **R-VAL-003** MUST: Field constraints (ge, le, min_length, max_length, regex, default_factory) MUST be applied to all fields with business rules or security requirements using Field().
- **R-VAL-004** MUST: No raw dictionary parameters are accepted in public service methods without prior Pydantic validation.
- **R-VAL-005** SHOULD: Use Field(default_factory=dict) for mutable defaults and Field(default_factory=lambda: datetime.now(timezone.utc)) for timestamps to prevent shared state bugs.
- **R-VAL-006** SHOULD: Implement @validator decorators for complex validation logic that cannot be expressed through Field constraints, and use @root_validator for cross-field validation.
- **R-VAL-007** SHOULD: Provide descriptive Field(description='...') annotations for all public API fields to support automatic OpenAPI documentation generation.
- **R-VAL-008** SHOULD: Use Optional[T] for nullable fields and provide sensible defaults where appropriate, avoiding Optional for required security-critical fields like user_id or event_id.
- **R-VAL-009** SHOULD: Define explicit Config classes with json_encoders for custom types (datetime: lambda v: v.isoformat()) to ensure consistent serialization across services.

### Verify

```bash
# Count BaseModel classes across service boundaries
grep -r 'class.*BaseModel' monorepo/tlt/mcp_services monorepo/tlt/agents monorepo/tlt/adapters --include='*.py' | wc -l

# Verify public request handlers use BaseModel
grep -r 'def.*request.*:' monorepo/tlt/mcp_services --include='*.py' -A 5 | grep -c 'BaseModel'

# Check for explicit type annotations in model definitions
python -c "import ast; import sys; [print(f'{n.name}: {any(b.id == \"BaseModel\" for b in n.bases if isinstance(b, ast.Name))}') for f in sys.argv[1:] for n in ast.walk(ast.parse(open(f).read())) if isinstance(n, ast.ClassDef)]" monorepo/tlt/mcp_services/*/models.py

# Detect raw dict usage in API boundaries (should return minimal results)
grep -r 'def.*request.*:.*dict' monorepo/tlt/mcp_services --include='*.py' | grep -v 'Dict\[' | grep -v '#'

# Verify Field constraints are applied
grep -r 'Field(' monorepo/tlt/mcp_services monorepo/tlt/agents --include='*.py' | grep -E '(ge=|le=|min_length|max_length|regex)' | wc -l
```

**Accept when:**
- All public API request/response models in mcp_services, agents, and adapters inherit from Pydantic BaseModel with explicit type annotations.
- Field constraints (ge, le, min_length, max_length, regex, default_factory) are applied to all fields with business rules or security requirements.
- No raw dictionary parameters are accepted in public service methods without prior Pydantic validation, verified by code review and linting.
- All security-critical fields (user_id, event_id, authentication tokens) use explicit types with no Any or implicit typing.
- Validation models include descriptive Field descriptions for OpenAPI documentation.

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks, CI pipeline checks, and code review are mandatory. Violations result in CI build failure. Security team review is required for any validation bypass or type: ignore comments in API boundary code.
</enforcement>