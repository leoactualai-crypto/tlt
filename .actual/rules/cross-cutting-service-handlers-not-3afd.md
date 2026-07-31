# Adopt Pydantic BaseModel for Domain Validation in Service Boundaries: Service Handlers Not

These rules are ALWAYS ACTIVE for all FastAPI service adapters, service endpoints, and domain validation logic crossing HTTP boundaries.

### Rules

- **R-PYDANTIC-001** MUST_NOT: Service handlers MUST NOT perform manual type coercion or validation that duplicates Pydantic's automatic validation.

### Verify

```bash
# Count BaseModel classes in codebase
grep -r 'class.*BaseModel' --include='*.py' | grep -v '__pycache__' | wc -l

# Verify response_model declarations on POST/PUT/PATCH endpoints
grep -r '@router\.(post|put|patch)' --include='*.py' | grep -c 'response_model='

# Extract and list all BaseModel subclasses in adapters and services
python -c "import ast; import sys; [print(f'{n.name}: {n.bases[0].id}') for f in sys.argv[1:] for n in ast.walk(ast.parse(open(f).read())) if isinstance(n, ast.ClassDef) and any(b.id == 'BaseModel' for b in n.bases if isinstance(b, ast.Name))]" $(find . -name '*.py' -path '*/adapters/*' -o -path '*/services/*')

# Flag manual type validation in handlers
grep -r 'isinstance\|type(' --include='*.py' | grep -E '(adapters|services)' | grep -v test | head -20
```

**Accept when:**
- All FastAPI endpoints in adapters/ and services/ directories declare request/response models using Pydantic BaseModel
- Grep for 'response_model=' in router decorators returns matches for at least 80% of POST/PUT/PATCH endpoints
- No manual type validation code (isinstance checks, type coercion) exists in endpoint handlers for fields already defined in Pydantic models
- Pydantic Field constraints (ge, le, description) are used to enforce business rules at the schema level
- Create/Response model separation is maintained to control field mutability

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All new FastAPI endpoints and service boundaries MUST declare Pydantic BaseModel for request/response validation. Manual type coercion in handlers duplicating Pydantic validation is a violation.
</enforcement>