# Adopt Pydantic BaseModel for Domain Validation in Service Boundaries: Fastapi Router Endpoints

These rules are ALWAYS ACTIVE for all FastAPI service adapters, service endpoints, and domain validation logic crossing HTTP boundaries.

### Rules

- **R-PYDANTIC-001** MUST: FastAPI router endpoints MUST declare response_model parameters using Pydantic BaseModel to establish explicit API contracts.
- **R-PYDANTIC-002** MUST: All FastAPI router endpoints accepting request bodies MUST use Pydantic BaseModel for input validation.
- **R-PYDANTIC-003** MUST: Domain entities used in service-to-service communication MUST be defined as Pydantic BaseModel classes.
- **R-PYDANTIC-004** SHOULD: Separate input models (Create, Update) from output models (Response) to control field mutability and prevent accidental exposure of internal fields.
- **R-PYDANTIC-005** SHOULD: Use Pydantic Field with ge/le constraints for numeric ranges (e.g., scores from 0.0 to 1.0) to enforce business rules at the schema level.
- **R-PYDANTIC-006** SHOULD: Use descriptive Field descriptions to document business rules and constraints for automatic API documentation.
- **R-PYDANTIC-007** MAY: For complex validation requiring multiple fields, implement custom validators using Pydantic's @validator or @root_validator decorators.

### Verify

```bash
# Count total BaseModel classes in codebase
grep -r 'class.*BaseModel' --include='*.py' | grep -v '__pycache__' | wc -l

# Check percentage of POST/PUT/PATCH endpoints with response_model declarations
grep -r '@router\.(post|put|patch)' --include='*.py' | grep -c 'response_model='

# List all BaseModel subclasses in adapters and services directories
python -c "import ast; import sys; [print(f'{n.name}: {n.bases[0].id}') for f in sys.argv[1:] for n in ast.walk(ast.parse(open(f).read())) if isinstance(n, ast.ClassDef) and any(b.id == 'BaseModel' for b in n.bases if isinstance(b, ast.Name))]" $(find . -name '*.py' -path '*/adapters/*' -o -path '*/services/*')
```

**Accept when:**
- All FastAPI endpoints in adapters/ and services/ directories declare request/response models using Pydantic BaseModel
- Grep for 'response_model=' in router decorators returns matches for at least 80% of POST/PUT/PATCH endpoints
- No manual type validation code (isinstance checks, type coercion) exists in endpoint handlers for fields already defined in Pydantic models
- Separate Create/Response model pairs are used for mutable input and immutable output contracts
- Numeric constraints use Pydantic Field ge/le parameters rather than imperative validation logic

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All new FastAPI endpoints and domain validation classes MUST comply with R-PYDANTIC-001 through R-PYDANTIC-007. CI pipeline and pre-commit hooks MUST enforce response_model declarations and flag manual type validation in handlers.
</enforcement>