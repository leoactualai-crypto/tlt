# Enforce Pydantic BaseModel Validation for Public API Contracts: Numeric Constraints Ranges

These rules are ALWAYS ACTIVE for all public/external API endpoints and data contracts within the monorepo/tlt system, including FastAPI routers in adapters (discord_adapter) and MCP service interfaces (vibe_bit, guild_manager, photo_vibe_check), and all inter-service boundaries where data contracts are exposed.

### Rules

- **R-PYDANTIC-001** MUST: Numeric constraints (ranges, bounds) MUST be enforced using Pydantic Field validators (ge, le, gt, lt) rather than runtime checks in all public API request/response models.
- **R-PYDANTIC-002** MUST: All FastAPI router endpoints with request bodies MUST use Pydantic BaseModel-derived classes for request/response models.
- **R-PYDANTIC-003** MUST: All MCP service models.py files MUST define API contracts using BaseModel with explicit Field constraints for validation.
- **R-PYDANTIC-004** SHOULD: Use pydantic.Field with descriptive descriptions for automatic OpenAPI documentation generation (e.g., Field(description='Photo quality score from 0 to 1')).
- **R-PYDANTIC-005** SHOULD: Leverage Field(default_factory=...) for datetime fields to avoid mutable default issues (e.g., Field(default_factory=lambda: datetime.now(timezone.utc))).
- **R-PYDANTIC-006** SHOULD: Define Enum classes for fields with fixed value sets and reference them in model fields to enforce valid options at validation time.
- **R-PYDANTIC-007** SHOULD: Use Optional[T] typing for truly optional fields and provide sensible defaults with Field(default=...) to improve API usability.
- **R-PYDANTIC-008** SHOULD: Implement custom validators using @validator decorators for cross-field validation or complex business rules that cannot be expressed with Field constraints.

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
- Numeric constraints on all public API models use Field validators (ge, le, gt, lt) rather than runtime checks

<enforcement>
Claude Code MUST NOT skip or defer verification. All new API endpoints and modifications to existing public API contracts MUST comply with R-PYDANTIC-001 through R-PYDANTIC-003 (MUST level rules). Violations block merge unless documented exceptions (EXC-001 for legacy migrations, EXC-002 for performance-critical paths) are approved by the architecture review board.
</enforcement>