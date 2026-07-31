# Adopt Pydantic BaseModel for Domain Validation in Service Boundaries: Domain Validation Classes

These rules are ALWAYS ACTIVE for all FastAPI service adapters, service endpoints, and domain validation logic crossing HTTP boundaries or service-to-service communication boundaries.

### Rules

- **R-PYDANTIC-001** MUST: Domain validation classes MUST use Pydantic BaseModel as the base class for all request/response models at service boundaries.
- **R-PYDANTIC-002** MUST: Domain validation classes MUST use Pydantic Field constraints (ge, le, min_length, max_length, regex) to enforce business rules at the schema level rather than in handler logic.
- **R-PYDANTIC-003** MUST: All FastAPI router endpoints accepting request bodies MUST declare request models using Pydantic BaseModel.
- **R-PYDANTIC-004** MUST: All FastAPI router endpoints returning structured responses MUST declare response_model parameters using Pydantic BaseModel.
- **R-PYDANTIC-005** SHOULD: Separate input models (Create, Update) from output models (Response) to control field mutability and prevent accidental exposure of internal fields.
- **R-PYDANTIC-006** SHOULD: Use descriptive Field descriptions to document business rules and constraints for automatic API documentation.
- **R-PYDANTIC-007** MAY: For complex validation requiring multiple fields, implement custom validators using Pydantic's @validator or @root_validator decorators.

### Verify

```bash
# Count BaseModel classes in codebase
grep -r 'class.*BaseModel' --include='*.py' | grep -v '__pycache__' | wc -l

# Verify response_model declarations on POST/PUT/PATCH endpoints
grep -r '@router\.(post|put|patch)' --include='*.py' | grep -c 'response_model='

# Extract and list all BaseModel subclasses with their names
python -c "import ast; import sys; [print(f'{n.name}: {n.bases[0].id}') for f in sys.argv[1:] for n in ast.walk(ast.parse(open(f).read())) if isinstance(n, ast.ClassDef) and any(b.id == 'BaseModel' for b in n.bases if isinstance(b, ast.Name))]" $(find . -name '*.py' -path '*/adapters/*' -o -path '*/services/*')

# Check for manual type validation in endpoint handlers
grep -r 'isinstance.*dict\|type(.*) ==' --include='*.py' | grep -v '__pycache__'
```

**Accept when:**
- All FastAPI endpoints in adapters/ and services/ directories declare request/response models using Pydantic BaseModel
- Grep for 'response_model=' in router decorators returns matches for at least 80% of POST/PUT/PATCH endpoints
- No manual type validation code (isinstance checks, type coercion) exists in endpoint handlers for fields already defined in Pydantic models
- All numeric range constraints use Field(ge=..., le=...) declarations
- Input and output models are clearly separated (e.g., ReminderCreate vs ReminderResponse)

<enforcement>
Claude Code MUST NOT skip or defer verification. All new FastAPI endpoints and domain validation classes MUST comply with R-PYDANTIC-001 through R-PYDANTIC-004 before merge. Code review MUST block violations of MUST-level rules. CI pipeline MUST fail if new endpoints lack response_model declarations.
</enforcement>