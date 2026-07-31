# Enforce Pydantic BaseModel Validation for Public API Contracts: Required Fields Declared

These rules are ALWAYS ACTIVE for all public/external API endpoints and data contracts within the monorepo/tlt system, including FastAPI routers in adapters (discord_adapter) and MCP service interfaces (vibe_bit, guild_manager, photo_vibe_check), and all inter-service boundaries.

### Rules

- **R-PYDANTIC-001** MUST: Required fields MUST be declared without Optional typing; optional fields MUST use Optional[T] or default values.
- **R-PYDANTIC-002** MUST: All FastAPI router endpoints with request bodies use Pydantic BaseModel-derived classes for request/response models.
- **R-PYDANTIC-003** MUST: All MCP service models.py files define API contracts using BaseModel with explicit Field constraints for validation.
- **R-PYDANTIC-004** MUST: Use pydantic.Field with descriptive descriptions for automatic OpenAPI documentation generation.
- **R-PYDANTIC-005** SHOULD: Leverage Field(default_factory=...) for datetime fields to avoid mutable default issues.
- **R-PYDANTIC-006** SHOULD: Define Enum classes for fields with fixed value sets and reference them in model fields to enforce valid options at validation time.
- **R-PYDANTIC-007** SHOULD: Implement custom validators using @validator decorators for cross-field validation or complex business rules that cannot be expressed with Field constraints.

### Verify

```bash
# Count BaseModel class definitions across adapters and MCP services
grep -r 'class.*BaseModel' monorepo/tlt/adapters/discord_adapter/*.py monorepo/tlt/mcp_services/*/models.py | wc -l

# Find router endpoints without BaseModel validation
grep -r '@router\.(post|put|patch)' monorepo/tlt/adapters/discord_adapter/*.py | xargs grep -L 'BaseModel'

# Verify all router endpoints reference BaseModel
python -c 'import ast; import sys; [sys.exit(1) for f in sys.argv[1:] if not any(isinstance(n, ast.ClassDef) and any(b.id == "BaseModel" for b in n.bases if isinstance(b, ast.Name)) for n in ast.walk(ast.parse(open(f).read())))]' monorepo/tlt/adapters/discord_adapter/*.py
```

**Accept when:**
- All FastAPI router endpoints with request bodies use Pydantic BaseModel-derived classes for request/response models
- All MCP service models.py files define API contracts using BaseModel with explicit Field constraints for validation
- No new API endpoints are merged that accept raw dictionaries or unvalidated input without documented exception approval
- Grep verification shows 100% of router endpoints with POST/PUT/PATCH methods reference BaseModel in the same file or imported models
- Required fields are declared without Optional typing; optional fields use Optional[T] or default values
- Field definitions include descriptive descriptions for OpenAPI documentation

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks, code review checklists, and CI pipeline static analysis MUST enforce this rule. Violations block merge until BaseModel validation is added or a documented exception (EXC-001 for legacy migrations or EXC-002 for performance-critical paths) is approved by the architecture review board.
</enforcement>