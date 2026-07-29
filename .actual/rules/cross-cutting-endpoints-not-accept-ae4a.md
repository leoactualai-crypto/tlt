# Adopt Pydantic BaseModel for Domain Validation in API Boundaries: Endpoints Not Accept

These rules are ALWAYS ACTIVE for all FastAPI router endpoints in adapters (discord_adapter, service APIs), domain models representing business entities (Reminder, Experience, Task, Photo analysis outputs), and request/response contracts for external HTTP clients at API ingress boundaries.

### Rules

- **R-PYDANTIC-001** MUST_NOT: Endpoints MUST NOT accept unvalidated dictionaries or primitive types for complex domain objects.

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
Claude Code MUST NOT skip or defer verification. All new API endpoints must declare Pydantic BaseModel-based validation models and response_model parameters. Violations block CI pipeline and require code review approval with documented exception rationale.
</enforcement>