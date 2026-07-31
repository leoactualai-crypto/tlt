# Adopt Pydantic BaseModel for Domain Validation in Service Boundaries: Fastapi Request Bodies

These rules are ALWAYS ACTIVE for all FastAPI service adapters, service endpoints, and domain validation logic that crosses HTTP boundaries or requires structured input validation and response serialization.

### Rules

- **R-PYDANTIC-001** MUST: All FastAPI request bodies and response models MUST be defined as Pydantic BaseModel subclasses with explicit field type annotations.
- **R-PYDANTIC-002** MUST: All FastAPI router endpoints accepting request bodies MUST declare request/response models using Pydantic BaseModel.
- **R-PYDANTIC-003** MUST: All FastAPI router endpoints returning structured responses MUST declare response_model parameters in router decorators.
- **R-PYDANTIC-004** SHOULD: Separate input models (Create, Update) from output models (Response) to control field mutability and prevent accidental exposure of internal fields.
- **R-PYDANTIC-005** SHOULD: Use Pydantic Field with ge/le constraints for numeric ranges (e.g., scores from 0.0 to 1.0) to enforce business rules at the schema level.
- **R-PYDANTIC-006** SHOULD: Use descriptive Field descriptions to document business rules and constraints for automatic API documentation.
- **R-PYDANTIC-007** MAY: For complex validation requiring multiple fields, implement custom validators using Pydantic's @validator or @root_validator decorators.

### Verify

```bash
# Count BaseModel subclasses in codebase
grep -r 'class.*BaseModel' --include='*.py' | grep -v '__pycache__' | wc -l

# Check response_model declarations on POST/PUT/PATCH endpoints
grep -r '@router\.(post|put|patch)' --include='*.py' | grep -c 'response_model='

# List all BaseModel classes in adapters and services
python -c "import ast; import sys; [print(f'{n.name}: {n.bases[0].id}') for f in sys.argv[1:] for n in ast.walk(ast.parse(open(f).read())) if isinstance(n, ast.ClassDef) and any(b.id == 'BaseModel' for b in n.bases if isinstance(b, ast.Name))]" $(find . -name '*.py' -path '*/adapters/*' -o -path '*/services/*')

# Verify no manual type validation in endpoint handlers
grep -r 'isinstance.*dict\|type(.*) ==' --include='*.py' | grep -v '__pycache__' | wc -l
```

**Accept when:**
- All FastAPI endpoints in adapters/ and services/ directories declare request/response models using Pydantic BaseModel
- Grep for 'response_model=' in router decorators returns matches for at least 80% of POST/PUT/PATCH endpoints
- No manual type validation code (isinstance checks, type coercion) exists in endpoint handlers for fields already defined in Pydantic models
- All domain entities crossing HTTP boundaries use explicit Pydantic field type annotations

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks running mypy type checking and CI pipeline grep checks verifying FastAPI endpoints declare response_model parameters are mandatory. Code review MUST block merge if validation logic duplicates Pydantic constraints or if new FastAPI endpoints lack response_model declarations.
</enforcement>