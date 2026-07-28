# Enforce Pydantic BaseModel for API Contract Validation in Public Interfaces: Models Provide Descriptive

These rules are ALWAYS ACTIVE for all public API contracts, service boundaries, and domain validation models across the codebase, including MCP services (gateway, rsvp, event_manager, photo_vibe_check, guild_manager), agents (ambient_event_agent), Discord adapters, authentication contexts, and external API request/response contracts.

### Rules

- **R-PYDANTIC-001** MUST: All public API request/response models in mcp_services, agents, and adapters inherit from Pydantic BaseModel with explicit type annotations.
- **R-PYDANTIC-002** MUST: Field constraints (ge, le, min_length, max_length, regex, default_factory) are applied to all fields with business rules or security requirements.
- **R-PYDANTIC-003** MUST: No raw dictionary parameters are accepted in public service methods without prior Pydantic validation.
- **R-PYDANTIC-004** SHOULD: Models SHOULD provide descriptive Field descriptions for API documentation and include examples in docstrings for complex validation rules.
- **R-PYDANTIC-005** SHOULD: Use Field(default_factory=dict) for mutable defaults and Field(default_factory=lambda: datetime.now(timezone.utc)) for timestamps to prevent shared state bugs.
- **R-PYDANTIC-006** SHOULD: Define explicit Config classes with json_encoders for custom types (datetime: lambda v: v.isoformat()) to ensure consistent serialization across services.
- **R-PYDANTIC-007** SHOULD: Implement @validator decorators for complex validation logic that cannot be expressed through Field constraints, and use @root_validator for cross-field validation.
- **R-PYDANTIC-008** SHOULD: Use Optional[T] for nullable fields and provide sensible defaults where appropriate, avoiding Optional for required security-critical fields like user_id or event_id.

### Verify

```bash
# Count BaseModel classes across service boundaries
grep -r 'class.*BaseModel' monorepo/tlt/mcp_services monorepo/tlt/agents monorepo/tlt/adapters --include='*.py' | wc -l

# Verify request handlers use BaseModel
grep -r 'def.*request.*:' monorepo/tlt/mcp_services --include='*.py' -A 5 | grep -c 'BaseModel'

# Check BaseModel inheritance in model files
python -c "import ast; import sys; [print(f'{n.name}: {any(b.id == \"BaseModel\" for b in n.bases if isinstance(b, ast.Name))}') for f in sys.argv[1:] for n in ast.walk(ast.parse(open(f).read())) if isinstance(n, ast.ClassDef)]" monorepo/tlt/mcp_services/*/models.py

# Detect raw dict usage in API boundaries
grep -r 'def.*request.*:.*dict' monorepo/tlt/mcp_services --include='*.py' | grep -v 'BaseModel'
```

**Accept when:**
- All public API request/response models in mcp_services, agents, and adapters inherit from Pydantic BaseModel with explicit type annotations.
- Field constraints (ge, le, min_length, max_length, regex, default_factory) are applied to all fields with business rules or security requirements.
- No raw dictionary parameters are accepted in public service methods without prior Pydantic validation, verified by code review and linting.
- All public API fields include descriptive Field(description='...') annotations for automatic OpenAPI documentation generation.

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks, CI pipeline checks, code review checklists, and automated security scanning are mandatory. Violations result in CI build failure, security team review, or architecture approval requirements. Exceptions require performance benchmarks or technical justification and must be re-justified quarterly.
</enforcement>