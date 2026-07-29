# Adopt Pydantic BaseModel for Domain Validation in API Boundaries: Field Level Validation

These rules are ALWAYS ACTIVE for all FastAPI router endpoints in adapters, domain models representing business entities, request/response contracts for external HTTP clients, and input validation at API ingress boundaries.

### Rules

- **R-PYDANTIC-001** SHOULD: Field-level validation constraints SHOULD use Pydantic Field with descriptive metadata (ge, le, description) for numeric ranges and bounded values.

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
Clause Code MUST NOT skip or defer verification. All new API endpoints MUST declare Pydantic BaseModel subclasses for request/response validation and include response_model in router decorators. CI pipeline MUST fail if these requirements are not met.
</enforcement>