# Enforce Pydantic BaseModel Validation for Public API Contracts: Public Response Models

These rules are ALWAYS ACTIVE for all public API response models, FastAPI router endpoints, MCP service interfaces, and inter-service data contracts within the monorepo/tlt system.

### Rules

- **R-PYDANTIC-001** MUST: All public API response models MUST inherit from `pydantic.BaseModel` to ensure consistent serialization and type safety.
- **R-PYDANTIC-002** MUST: All FastAPI router endpoints in `monorepo/tlt/adapters/discord_adapter` (event.py, rsvp.py, reminder.py) MUST use Pydantic BaseModel-derived classes for request/response models.
- **R-PYDANTIC-003** MUST: All MCP service models in `monorepo/tlt/mcp_services` (vibe_bit/models.py, guild_manager/models.py, photo_vibe_check/photo_processor.py) MUST define API contracts using BaseModel with explicit Field constraints for validation.
- **R-PYDANTIC-004** MUST: All service-to-service contract definitions used in inter-service communication MUST use Pydantic BaseModel validation.
- **R-PYDANTIC-005** MUST: All data models exposed through HTTP APIs, CloudEvents endpoints, or agent task interfaces MUST inherit from BaseModel.
- **R-PYDANTIC-006** SHOULD: Use `pydantic.Field` with descriptive descriptions for automatic OpenAPI documentation generation (e.g., `Field(description='Photo quality score from 0 to 1')`).
- **R-PYDANTIC-007** SHOULD: Leverage `Field(default_factory=...)` for datetime fields to avoid mutable default issues (e.g., `Field(default_factory=lambda: datetime.now(timezone.utc))`).
- **R-PYDANTIC-008** SHOULD: Define Enum classes for fields with fixed value sets and reference them in model fields to enforce valid options at validation time.
- **R-PYDANTIC-009** SHOULD: Use `Optional[T]` typing for truly optional fields and provide sensible defaults with `Field(default=...)` to improve API usability.
- **R-PYDANTIC-010** SHOULD: Implement custom validators using `@validator` decorators for cross-field validation or complex business rules that cannot be expressed with Field constraints.

### Verify

```bash
# Count BaseModel class definitions in adapter and MCP service files
grep -r 'class.*BaseModel' monorepo/tlt/adapters/discord_adapter/*.py monorepo/tlt/mcp_services/*/models.py | wc -l

# Find POST/PUT/PATCH router endpoints without BaseModel references
grep -r '@router\.(post|put|patch)' monorepo/tlt/adapters/discord_adapter/*.py | xargs grep -L 'BaseModel'

# Verify all router endpoint files contain BaseModel class definitions
python -c 'import ast; import sys; [sys.exit(1) for f in sys.argv[1:] if not any(isinstance(n, ast.ClassDef) and any(b.id == "BaseModel" for b in n.bases if isinstance(b, ast.Name)) for n in ast.walk(ast.parse(open(f).read())))]' monorepo/tlt/adapters/discord_adapter/*.py
```

**Accept when:**
- All FastAPI router endpoints with request bodies use Pydantic BaseModel-derived classes for request/response models
- All MCP service models.py files define API contracts using BaseModel with explicit Field constraints for validation
- No new API endpoints are merged that accept raw dictionaries or unvalidated input without documented exception approval
- Grep verification shows 100% of router endpoints with POST/PUT/PATCH methods reference BaseModel in the same file or imported models
- All public response models inherit from `pydantic.BaseModel` with no exceptions in scope

<enforcement>
Claude Code MUST NOT skip or defer verification. All new API endpoints and response models MUST be checked against these rules before approval. CI build MUST fail if new API endpoints lack Pydantic validation without documented exception (EXC-001 or EXC-002). Code review MUST block merge until BaseModel validation is added or exception is approved by architecture review board.
</enforcement>