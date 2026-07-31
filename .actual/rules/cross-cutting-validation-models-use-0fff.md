# Adopt Pydantic BaseModel for Domain Validation in Service Boundaries: Validation Models Use

These rules are ALWAYS ACTIVE for all FastAPI service adapters, service endpoints, and domain validation logic crossing HTTP boundaries or service-to-service communication boundaries.

### Rules

- **R-PYDANTIC-001** MUST: All FastAPI router endpoints accepting request bodies SHALL declare request models using Pydantic BaseModel.
- **R-PYDANTIC-002** MUST: All FastAPI router endpoints returning structured responses SHALL declare response_model parameters using Pydantic BaseModel on router decorators.
- **R-PYDANTIC-003** MUST: Domain entities used in service-to-service communication SHALL be defined as Pydantic BaseModel classes with explicit field types and constraints.
- **R-PYDANTIC-004** SHOULD: Separate input models (Create, Update) from output models (Response) to control field mutability and prevent accidental exposure of internal fields.
- **R-PYDANTIC-005** SHOULD: Use Pydantic Field with ge/le constraints for numeric ranges (e.g., scores from 0.0 to 1.0) to enforce business rules at the schema level.
- **R-PYDANTIC-006** SHOULD: Use descriptive Field descriptions to document business rules and constraints for automatic API documentation.
- **R-PYDANTIC-007** MAY: Validation models MAY use Pydantic validators or root_validators for complex cross-field validation logic that cannot be expressed through Field constraints.
- **R-PYDANTIC-008** MUST NOT: Manual type validation code (isinstance checks, type coercion) SHALL NOT exist in endpoint handlers for fields already defined in Pydantic models.

### Verify

```bash
# Count BaseModel classes in codebase
grep -r 'class.*BaseModel' --include='*.py' | grep -v '__pycache__' | wc -l

# Verify response_model declarations on POST/PUT/PATCH endpoints
grep -r '@router\.(post|put|patch)' --include='*.py' | grep -c 'response_model='

# Extract and list all BaseModel subclasses with their names
python -c "import ast; import sys; [print(f'{n.name}: {n.bases[0].id}') for f in sys.argv[1:] for n in ast.walk(ast.parse(open(f).read())) if isinstance(n, ast.ClassDef) and any(b.id == 'BaseModel' for b in n.bases if isinstance(b, ast.Name))]" $(find . -name '*.py' -path '*/adapters/*' -o -path '*/services/*')

# Check for manual type validation in endpoint handlers
grep -r 'isinstance.*dict\|type(.*) ==' --include='*.py' | grep -E '(adapters|services)' | grep -v test
```

**Accept when:**
- All FastAPI endpoints in adapters/ and services/ directories declare request/response models using Pydantic BaseModel
- Grep for 'response_model=' in router decorators returns matches for at least 80% of POST/PUT/PATCH endpoints
- No manual type validation code (isinstance checks, type coercion) exists in endpoint handlers for fields already defined in Pydantic models
- Numeric constraints use Field(ge=..., le=...) declarations rather than imperative validation logic
- Input and output models are clearly separated (e.g., ReminderCreate vs ReminderResponse)

<enforcement>
Claude Code MUST NOT skip or defer verification. All new FastAPI endpoints and service boundary definitions MUST be checked against these rules before acceptance. CI build MUST fail if new FastAPI endpoints lack response_model declarations. Code review MUST block merge if validation logic duplicates Pydantic constraints or manual type validation exists in handlers.
</enforcement>