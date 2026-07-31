# Enforce Pydantic BaseModel Validation for Public API Contracts: Enum Types Used

These rules are ALWAYS ACTIVE for all public/external API endpoints and data contracts within the monorepo/tlt system, including FastAPI routers in adapters (discord_adapter) and MCP service interfaces (vibe_bit, guild_manager, photo_vibe_check), and all inter-service boundaries.

### Rules

- **R-PYDANTIC-ENUM-001** SHOULD: Enum types SHOULD be used for fields with fixed value sets (e.g., ElementType, PhotoQuality) to enforce valid options at the API boundary.

### Verify

```bash
# Count BaseModel classes in adapter and MCP service files
grep -r 'class.*BaseModel' monorepo/tlt/adapters/discord_adapter/*.py monorepo/tlt/mcp_services/*/models.py | wc -l

# Find router endpoints without BaseModel validation
grep -r '@router\.(post|put|patch)' monorepo/tlt/adapters/discord_adapter/*.py | xargs grep -L 'BaseModel'

# Verify all router endpoints reference BaseModel in same file or imports
python -c 'import ast; import sys; [sys.exit(1) for f in sys.argv[1:] if not any(isinstance(n, ast.ClassDef) and any(b.id == "BaseModel" for b in n.bases if isinstance(b, ast.Name)) for n in ast.walk(ast.parse(open(f).read())))]' monorepo/tlt/adapters/discord_adapter/*.py
```

**Accept when:**
- All FastAPI router endpoints with request bodies use Pydantic BaseModel-derived classes for request/response models
- All MCP service models.py files define API contracts using BaseModel with explicit Field constraints for validation
- No new API endpoints are merged that accept raw dictionaries or unvalidated input without documented exception approval
- Grep verification shows 100% of router endpoints with POST/PUT/PATCH methods reference BaseModel in the same file or imported models
- Enum types are defined for fields with fixed value sets and referenced in model fields to enforce valid options at validation time

<enforcement>
Claude Code MUST NOT skip or defer verification. All new API endpoints must use Pydantic BaseModel with Enum types for constrained fields, or document an approved exception (EXC-001 for legacy migration or EXC-002 for performance-critical paths).
</enforcement>