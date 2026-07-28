# Enforce Pydantic BaseModel for API Contract Validation in Public Interfaces: Input Validation Models

These rules are ALWAYS ACTIVE for all public API contracts, service boundaries, and domain validation models across the codebase, including MCP services (gateway, rsvp, event_manager, photo_vibe_check, guild_manager), agents (ambient_event_agent), Discord adapters, and authentication/authorization contexts.

### Rules

- **R-VAL-001** MUST: Input validation models MUST use Pydantic BaseModel as the base class for all public API request/response models, service boundary contracts, and domain entities processing external input.
- **R-VAL-002** MUST: Input validation models MUST use Pydantic Field constraints (ge, le, min_length, max_length, regex, default_factory) to enforce business rules and security boundaries at the schema level.
- **R-VAL-003** MUST: All public service methods accepting external input MUST require Pydantic BaseModel instances as parameters, never raw dictionaries or unvalidated data structures.
- **R-VAL-004** MUST: Field constraints MUST be applied to all fields with business rules or security requirements (e.g., quality_score: float = Field(ge=0.0, le=1.0), emoji: str = Field(regex=r'^\p{Emoji}$')).
- **R-VAL-005** SHOULD: Complex validation logic that cannot be expressed through Field constraints SHOULD use @validator decorators, and cross-field validation SHOULD use @root_validator.
- **R-VAL-006** SHOULD: All public API fields SHOULD include descriptive Field(description='...') annotations to support automatic OpenAPI documentation generation.
- **R-VAL-007** SHOULD: Mutable defaults SHOULD use Field(default_factory=dict) and timestamps SHOULD use Field(default_factory=lambda: datetime.now(timezone.utc)) to prevent shared state bugs.
- **R-VAL-008** SHOULD: Config classes with json_encoders SHOULD be defined for custom types (e.g., datetime: lambda v: v.isoformat()) to ensure consistent serialization across services.

### Verify

```bash
# Count BaseModel classes in service, agent, and adapter modules
grep -r 'class.*BaseModel' monorepo/tlt/mcp_services monorepo/tlt/agents monorepo/tlt/adapters --include='*.py' | wc -l

# Verify public request handlers use BaseModel
grep -r 'def.*request.*:' monorepo/tlt/mcp_services --include='*.py' -A 5 | grep -c 'BaseModel'

# Check BaseModel inheritance in all models.py files
python -c "import ast; import sys; [print(f'{n.name}: {any(b.id == \"BaseModel\" for b in n.bases if isinstance(b, ast.Name))}') for f in sys.argv[1:] for n in ast.walk(ast.parse(open(f).read())) if isinstance(n, ast.ClassDef)]" monorepo/tlt/mcp_services/*/models.py

# Detect raw dict usage in public API boundaries
grep -r 'def.*request.*:.*dict' monorepo/tlt/mcp_services --include='*.py' | grep -v 'BaseModel'

# Verify Field constraints are applied
grep -r 'Field(' monorepo/tlt/mcp_services monorepo/tlt/agents --include='*.py' | grep -E '(ge=|le=|min_length=|max_length=|regex=)' | wc -l
```

**Accept when:**
- All public API request/response models in mcp_services, agents, and adapters inherit from Pydantic BaseModel with explicit type annotations.
- Field constraints (ge, le, min_length, max_length, regex, default_factory) are applied to all fields with business rules or security requirements.
- No raw dictionary parameters are accepted in public service methods without prior Pydantic validation, verified by code review and linting.
- All public API fields include descriptive Field(description='...') annotations or have clear inline documentation.
- Complex validation logic uses @validator or @root_validator decorators appropriately.

<enforcement>
Claude Code MUST NOT skip or defer verification. All new public API endpoints and external input handlers MUST use Pydantic BaseModel validation models. CI build MUST fail if validation models are missing. Security team review is REQUIRED for any validation bypass or type: ignore comments in API boundary code.
</enforcement>