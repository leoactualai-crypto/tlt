# Adopt Pydantic BaseModel for Domain Validation in Service Boundaries: Complex Domain Entities

These rules are ALWAYS ACTIVE for all FastAPI service adapters, service endpoints, and domain validation logic that crosses HTTP boundaries or service-to-service communication boundaries.

### Rules

- **R-PYDANTIC-001** SHOULD: Complex domain entities SHOULD separate input models (Create, Update) from output models (Response) to distinguish between mutable and immutable fields.

### Verify

```bash
# Count BaseModel classes in the codebase
grep -r 'class.*BaseModel' --include='*.py' | grep -v '__pycache__' | wc -l

# Verify response_model declarations on FastAPI endpoints
grep -r '@router\.(post|put|patch)' --include='*.py' | grep -c 'response_model='

# Extract and list all BaseModel subclasses in adapters and services
python -c "import ast; import sys; [print(f'{n.name}: {n.bases[0].id}') for f in sys.argv[1:] for n in ast.walk(ast.parse(open(f).read())) if isinstance(n, ast.ClassDef) and any(b.id == 'BaseModel' for b in n.bases if isinstance(b, ast.Name))]" $(find . -name '*.py' -path '*/adapters/*' -o -path '*/services/*')
```

**Accept when:**
- All FastAPI endpoints in adapters/ and services/ directories declare request/response models using Pydantic BaseModel
- Grep for 'response_model=' in router decorators returns matches for at least 80% of POST/PUT/PATCH endpoints
- No manual type validation code (isinstance checks, type coercion) exists in endpoint handlers for fields already defined in Pydantic models
- Input models (Create, Update) and output models (Response) are clearly separated for complex domain entities
- Field-level constraints (ge, le, description) are used to enforce business rules at the schema level

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks and CI pipeline checks MUST validate that all new FastAPI endpoints declare response_model parameters and use Pydantic BaseModel for validation. Code review MUST block merges that duplicate validation logic already defined in Pydantic constraints.
</enforcement>