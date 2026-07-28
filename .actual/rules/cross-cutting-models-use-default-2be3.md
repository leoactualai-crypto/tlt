# Enforce Pydantic BaseModel for API Contract Validation in Public Interfaces: Models Use Default

These rules are ALWAYS ACTIVE for all public API contracts, service boundaries, and domain validation models across MCP services (gateway, rsvp, event_manager, photo_vibe_check, guild_manager), agents (ambient_event_agent), Discord adapters, authentication contexts, and external API request/response contracts.

### Rules

- **R-PYDANTIC-001** MUST: All public API request/response models inherit from Pydantic BaseModel with explicit type annotations.
- **R-PYDANTIC-002** SHOULD: Models SHOULD use `default_factory` for mutable defaults (dict, list) to prevent shared state bugs and use `Field(default_factory=lambda: datetime.now(timezone.utc))` for timestamps.
- **R-PYDANTIC-003** SHOULD: Field constraints (ge, le, min_length, max_length, regex) SHOULD be applied to all fields with business rules or security requirements.
- **R-PYDANTIC-004** MUST: No raw dictionary parameters are accepted in public service methods without prior Pydantic validation.
- **R-PYDANTIC-005** SHOULD: Descriptive `Field(description='...')` annotations SHOULD be provided for all public API fields to support automatic OpenAPI documentation generation.

### Verify

```bash
# Count BaseModel classes across service boundaries
grep -r 'class.*BaseModel' monorepo/tlt/mcp_services monorepo/tlt/agents monorepo/tlt/adapters --include='*.py' | wc -l

# Verify request handlers use BaseModel
grep -r 'def.*request.*:' monorepo/tlt/mcp_services --include='*.py' -A 5 | grep -c 'BaseModel'

# Check BaseModel inheritance in model files
python -c "import ast; import sys; [print(f'{n.name}: {any(b.id == \"BaseModel\" for b in n.bases if isinstance(b, ast.Name))}') for f in sys.argv[1:] for n in ast.walk(ast.parse(open(f).read())) if isinstance(n, ast.ClassDef)]" monorepo/tlt/mcp_services/*/models.py

# Detect raw dict usage in API boundaries
grep -r 'def.*request.*:.*dict' monorepo/tlt/mcp_services --include='*.py'

# Verify Field constraints are applied
grep -r 'Field(' monorepo/tlt/mcp_services monorepo/tlt/agents --include='*.py' | grep -E '(ge=|le=|min_length|max_length|regex|default_factory)' | wc -l
```

**Accept when:**
- All public API request/response models in mcp_services, agents, and adapters inherit from Pydantic BaseModel with explicit type annotations.
- Mutable defaults use `default_factory` (e.g., `Field(default_factory=dict)`) and timestamps use `Field(default_factory=lambda: datetime.now(timezone.utc))`.
- Field constraints (ge, le, min_length, max_length, regex) are applied to all fields with business rules or security requirements.
- No raw dictionary parameters are accepted in public service methods without prior Pydantic validation.
- Descriptive Field descriptions are provided for all public API fields.
- Code review confirms no validation bypasses or `type: ignore` comments in API boundary code.

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks, CI pipeline checks, and code review are mandatory before merging changes to public API models. Violations result in build failure and require security team review.
</enforcement>