# Enforce Pydantic BaseModel for API Contract Validation in Public Interfaces: Public Contracts Request

These rules are ALWAYS ACTIVE for all public API contracts, service boundaries, and domain validation models across the codebase, including all MCP services (gateway, rsvp, event_manager, photo_vibe_check, guild_manager), agents (ambient_event_agent), Discord adapters, authentication contexts, and external API request/response contracts.

### Rules

- **R-PYD-001** MUST: All public API contracts, request/response models, and domain entities MUST inherit from Pydantic BaseModel to enforce runtime type validation and constraint checking.
- **R-PYD-002** MUST: Apply Field constraints (ge, le, min_length, max_length, regex, default_factory) to all fields with business rules or security requirements.
- **R-PYD-003** MUST: Use Field(default_factory=dict) for mutable defaults and Field(default_factory=lambda: datetime.now(timezone.utc)) for timestamps to prevent shared state bugs.
- **R-PYD-004** MUST: No raw dictionary parameters are accepted in public service methods without prior Pydantic validation.
- **R-PYD-005** SHOULD: Implement @validator decorators for complex validation logic that cannot be expressed through Field constraints, and use @root_validator for cross-field validation.
- **R-PYD-006** SHOULD: Provide descriptive Field(description='...') annotations for all public API fields to support automatic OpenAPI documentation generation.
- **R-PYD-007** SHOULD: Use Optional[T] for nullable fields and provide sensible defaults where appropriate, avoiding Optional for required security-critical fields like user_id or event_id.
- **R-PYD-008** SHOULD: Define explicit Config classes with json_encoders for custom types (datetime: lambda v: v.isoformat()) to ensure consistent serialization across services.

### Verify

```bash
# Count BaseModel inheritance across service boundaries
grep -r 'class.*BaseModel' monorepo/tlt/mcp_services monorepo/tlt/agents monorepo/tlt/adapters --include='*.py' | wc -l

# Verify request handlers use BaseModel
grep -r 'def.*request.*:' monorepo/tlt/mcp_services --include='*.py' -A 5 | grep -c 'BaseModel'

# Analyze class definitions for BaseModel inheritance
python -c "import ast; import sys; [print(f'{n.name}: {any(b.id == \"BaseModel\" for b in n.bases if isinstance(b, ast.Name))}') for f in sys.argv[1:] for n in ast.walk(ast.parse(open(f).read())) if isinstance(n, ast.ClassDef)]" monorepo/tlt/mcp_services/*/models.py

# Detect raw dict usage in public methods (security check)
grep -r 'def.*request.*:.*dict' monorepo/tlt/mcp_services --include='*.py' | grep -v 'BaseModel'
```

**Accept when:**
- All public API request/response models in mcp_services, agents, and adapters inherit from Pydantic BaseModel with explicit type annotations.
- Field constraints (ge, le, min_length, max_length, regex, default_factory) are applied to all fields with business rules or security requirements.
- No raw dictionary parameters are accepted in public service methods without prior Pydantic validation, verified by code review and linting.
- All public API fields include descriptive Field(description='...') annotations for OpenAPI documentation.

<enforcement>
Claude Code MUST NOT skip or defer verification. All new public API contracts and domain models must be validated against these rules before acceptance. CI build fails if new API endpoints are added without Pydantic validation models. Security team review is required for any validation bypass or type: ignore comments in API boundary code.
</enforcement>