# Adopt Pydantic BaseModel for Domain Validation in API Boundaries: Domain Validation Models

These rules are ALWAYS ACTIVE for all FastAPI router endpoints in adapters, domain models representing business entities, request/response contracts for external HTTP clients, and input validation at API ingress boundaries.

### Rules

- **R-PYDANTIC-001** MUST: Domain validation models MUST define explicit field types with constraints using Pydantic Field validators where applicable (e.g., ge, le, description).

### Verify

```bash
# Count BaseModel subclasses in adapter and service directories
grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/adapters monorepo/tlt/services monorepo/tlt/mcp_services | wc -l

# Count router endpoints with response_model declarations
grep -r '@router\.(get|post|put|delete).*response_model=' --include='*.py' monorepo/tlt | wc -l

# Check if a specific file contains BaseModel subclass
python -c 'import ast; import sys; tree = ast.parse(open(sys.argv[1]).read()); print(any(base.id == "BaseModel" for node in ast.walk(tree) if isinstance(node, ast.ClassDef) for base in node.bases if hasattr(base, "id")))' <file_path>
```

**Accept when:**
- All FastAPI endpoint files contain at least one BaseModel subclass for request or response validation
- All router endpoint decorators with complex request/response bodies declare response_model parameter
- Grep for 'class.*BaseModel' in API adapter directories returns count matching number of domain validation models

<enforcement>
Claude Code MUST NOT skip or defer verification. All new API endpoints must include Pydantic BaseModel validation models with explicit field types and constraints. Code review must enforce response_model declarations on all router decorators. CI pipeline must fail if validation models use untyped dictionaries or endpoints lack response_model declarations.
</enforcement>