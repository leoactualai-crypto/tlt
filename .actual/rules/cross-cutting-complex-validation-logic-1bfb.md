# Enforce Pydantic BaseModel Validation for Public API Contracts: Complex Validation Logic

These rules are ALWAYS ACTIVE for all public/external API endpoints and data contracts within the monorepo/tlt system, including FastAPI routers in adapters (discord_adapter) and MCP service interfaces (vibe_bit, guild_manager, photo_vibe_check), and all inter-service boundaries where data contracts are exposed.

### Rules

- **R-PYDANTIC-001** SHOULD: Complex validation logic SHOULD use Pydantic validators (@validator, @root_validator) to keep validation logic co-located with models.
- **R-PYDANTIC-002** MUST: All FastAPI router endpoints with request bodies use Pydantic BaseModel-derived classes for request/response models.
- **R-PYDANTIC-003** MUST: All MCP service models.py files define API contracts using BaseModel with explicit Field constraints for validation.
- **R-PYDANTIC-004** SHOULD: Use pydantic.Field with descriptive descriptions for automatic OpenAPI documentation generation.
- **R-PYDANTIC-005** SHOULD: Leverage Field(default_factory=...) for datetime fields to avoid mutable default issues.
- **R-PYDANTIC-006** SHOULD: Define Enum classes for fields with fixed value sets and reference them in model fields to enforce valid options at validation time.
- **R-PYDANTIC-007** SHOULD: Use Optional[T] typing for truly optional fields and provide sensible defaults with Field(default=...) to improve API usability.
- **R-PYDANTIC-008** SHOULD: Implement custom validators using @validator decorators for cross-field validation or complex business rules that cannot be expressed with Field constraints.

### Verify

```bash
# Count BaseModel class definitions across adapters and MCP services
grep -r 'class.*BaseModel' monorepo/tlt/adapters/discord_adapter/*.py monorepo/tlt/mcp_services/*/models.py | wc -l

# Find router endpoints without BaseModel validation
grep -r '@router\.(post|put|patch)' monorepo/tlt/adapters/discord_adapter/*.py | xargs grep -L 'BaseModel'

# Verify all API contract files use BaseModel
python -c 'import ast; import sys; [sys.exit(1) for f in sys.argv[1:] if not any(isinstance(n, ast.ClassDef) and any(b.id == "BaseModel" for b in n.bases if isinstance(b, ast.Name)) for n in ast.walk(ast.parse(open(f).read())))]' monorepo/tlt/adapters/discord_adapter/*.py
```

**Accept when:**
- All FastAPI router endpoints with request bodies use Pydantic BaseModel-derived classes for request/response models
- All MCP service models.py files define API contracts using BaseModel with explicit Field constraints for validation
- No new API endpoints are merged that accept raw dictionaries or unvalidated input without documented exception approval
- Grep verification shows 100% of router endpoints with POST/PUT/PATCH methods reference BaseModel in the same file or imported models
- Complex validation logic uses @validator or @root_validator decorators rather than external validation functions
- Field definitions include descriptive descriptions for OpenAPI documentation
- Datetime fields use Field(default_factory=...) to avoid mutable defaults
- Enum classes are defined for fixed value sets and referenced in model fields

<enforcement>
Claude Code MUST NOT skip or defer verification. All new API endpoints and service contracts MUST use Pydantic BaseModel validation. Code review MUST block merge until BaseModel validation is added or a documented exception (EXC-001 or EXC-002) is approved. CI pipeline MUST fail if new API endpoints lack Pydantic validation without documented exception.
</enforcement>