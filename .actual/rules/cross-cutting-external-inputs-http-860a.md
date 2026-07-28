# Enforce Pydantic BaseModel for API Contract Validation in Public Interfaces: External Inputs Http

These rules are ALWAYS ACTIVE for all public API contracts, service boundaries, and domain validation models across the codebase, including MCP services (gateway, rsvp, event_manager, photo_vibe_check, guild_manager), agents (ambient_event_agent), Discord adapters, and authentication/authorization contexts.

### Rules

- **R-PYDANTIC-001** MUST: All external inputs from HTTP requests, CloudEvents, Discord messages, and inter-service calls MUST be validated through Pydantic BaseModel before processing.
- **R-PYDANTIC-002** MUST: All public API request/response models inherit from Pydantic BaseModel with explicit type annotations.
- **R-PYDANTIC-003** MUST: Field constraints (ge, le, min_length, max_length, regex, default_factory) are applied to all fields with business rules or security requirements.
- **R-PYDANTIC-004** MUST: No raw dictionary parameters are accepted in public service methods without prior Pydantic validation.
- **R-PYDANTIC-005** SHOULD: Use Field(default_factory=dict) for mutable defaults and Field(default_factory=lambda: datetime.now(timezone.utc)) for timestamps to prevent shared state bugs.
- **R-PYDANTIC-006** SHOULD: Apply @validator decorators for complex validation logic that cannot be expressed through Field constraints, and use @root_validator for cross-field validation.
- **R-PYDANTIC-007** SHOULD: Provide descriptive Field(description='...') annotations for all public API fields to support automatic OpenAPI documentation generation.
- **R-PYDANTIC-008** SHOULD: Use Optional[T] for nullable fields and provide sensible defaults where appropriate, avoiding Optional for required security-critical fields like user_id or event_id.

### Verify

```bash
# Count BaseModel classes in service, agent, and adapter modules
grep -r 'class.*BaseModel' monorepo/tlt/mcp_services monorepo/tlt/agents monorepo/tlt/adapters --include='*.py' | wc -l

# Verify request handlers use BaseModel validation
grep -r 'def.*request.*:' monorepo/tlt/mcp_services --include='*.py' -A 5 | grep -c 'BaseModel'

# Check for BaseModel inheritance in model files
python -c "import ast; import sys; [print(f'{n.name}: {any(b.id == \"BaseModel\" for b in n.bases if isinstance(b, ast.Name))}') for f in sys.argv[1:] for n in ast.walk(ast.parse(open(f).read())) if isinstance(n, ast.ClassDef)]" monorepo/tlt/mcp_services/*/models.py

# Detect raw dict usage in request handlers (potential validation bypass)
grep -r 'def.*request.*:' monorepo/tlt/mcp_services --include='*.py' -A 10 | grep -E '(dict\[|Dict\[|: dict)' | grep -v BaseModel
```

**Accept when:**
- All public API request/response models in mcp_services, agents, and adapters inherit from Pydantic BaseModel with explicit type annotations.
- Field constraints (ge, le, min_length, max_length, regex, default_factory) are applied to all fields with business rules or security requirements.
- No raw dictionary parameters are accepted in public service methods without prior Pydantic validation, verified by code review and linting.
- Pre-commit hooks and CI pipeline checks confirm all models.py files contain BaseModel inheritance and Field constraints.
- Code review checklist confirms validation model review for all new API endpoints and external input handlers.

<enforcement>
Claude Code MUST NOT skip or defer verification. All external inputs at service boundaries MUST be validated through Pydantic BaseModel. Violations require security team review and architecture approval before merge.
</enforcement>