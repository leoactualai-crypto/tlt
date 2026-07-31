# Adopt Pydantic BaseModel for Domain Validation in Service Boundaries: Pydantic Models Include

These rules are ALWAYS ACTIVE for all FastAPI service adapters, service endpoints, and domain validation logic crossing HTTP boundaries.

### Rules

- **R-PYDANTIC-001** SHOULD: Pydantic models SHOULD include Field descriptions to document the purpose and constraints of each field for API documentation.

### Verify

```bash
# Count BaseModel classes in codebase
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
- Pydantic Field constraints (ge, le, description) are used to enforce business rules at the schema level
- Create/Response model separation is implemented to establish clear boundaries between mutable input and immutable output

<enforcement>
Claude Code MUST NOT skip or defer verification. All new FastAPI endpoints MUST declare response_model parameters using Pydantic BaseModel. Code review MUST block merges if validation logic duplicates Pydantic constraints or if manual type coercion exists in handlers.
</enforcement>