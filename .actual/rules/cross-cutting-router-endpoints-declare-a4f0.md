# Adopt Pydantic BaseModel for Domain Validation in API Boundaries: Router Endpoints Declare

These rules are ALWAYS ACTIVE for all FastAPI router endpoints in adapters and service APIs that define request/response contracts at API boundaries.

### Rules

- **R-PYDANTIC-001** MUST: API router endpoints MUST declare `response_model` parameter using Pydantic BaseModel subclasses to enforce response schema validation.

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
- All FastAPI endpoint files in adapter directories contain at least one BaseModel subclass for request or response validation
- All router endpoint decorators with complex request/response bodies declare `response_model` parameter
- Grep for `class.*BaseModel` in API adapter directories returns a count matching the number of domain validation models

<enforcement>
Clause Code MUST NOT skip or defer verification of R-PYDANTIC-001 compliance. All new router endpoints must declare response_model with Pydantic BaseModel subclasses. CI pipeline must fail if violations are detected.
</enforcement>