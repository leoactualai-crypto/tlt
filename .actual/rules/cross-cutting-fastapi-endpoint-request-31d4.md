# Adopt Pydantic BaseModel for Domain Validation in API Boundaries: Fastapi Endpoint Request

These rules are ALWAYS ACTIVE for all FastAPI endpoint request and response models, domain models representing business entities, and API ingress boundaries within adapter and service layers.

### Rules

- **R-PYDANTIC-001** MUST: All FastAPI endpoint request and response models MUST inherit from `pydantic.BaseModel`.
- **R-PYDANTIC-002** MUST: All router endpoint decorators with complex request/response bodies MUST declare the `response_model` parameter.
- **R-PYDANTIC-003** MUST: Domain models representing business entities (Reminder, Experience, Task, Photo analysis outputs) MUST use Pydantic BaseModel for API contracts.
- **R-PYDANTIC-004** SHOULD: Separate input models (Create, Update) from output models (Response) to distinguish validation concerns.
- **R-PYDANTIC-005** SHOULD: Use `Field()` for field-level constraints: `rating: int = Field(ge=1, le=5, description='Rating from 1 to 5')`.
- **R-PYDANTIC-006** SHOULD: Use `Optional[Type]` with default `None` for nullable fields: `photos: Optional[List[str]] = None`.
- **R-PYDANTIC-007** MAY: Legacy endpoints undergoing migration may temporarily use dict-based validation (EXC-001), with documented migration plan and tech lead approval.

### Verify

```bash
# Count BaseModel subclasses in adapter and service directories
grep -r 'class.*BaseModel' --include='*.py' monorepo/tlt/adapters monorepo/tlt/services monorepo/tlt/mcp_services | wc -l

# Count router endpoints with response_model declarations
grep -r '@router\.(get|post|put|delete).*response_model=' --include='*.py' monorepo/tlt | wc -l

# Verify BaseModel inheritance in specific file
python -c 'import ast; import sys; tree = ast.parse(open(sys.argv[1]).read()); print(any(base.id == "BaseModel" for node in ast.walk(tree) if isinstance(node, ast.ClassDef) for base in node.bases if hasattr(base, "id")))' <file_path>
```

**Accept when:**
- All FastAPI endpoint files in adapter directories contain at least one BaseModel subclass for request or response validation.
- All router endpoint decorators with complex request/response bodies declare the `response_model` parameter.
- Grep for `class.*BaseModel` in API adapter directories returns a count matching the number of domain validation models.
- Type annotations on all model fields pass mypy static analysis.

<enforcement>
Clause Code MUST NOT skip or defer verification. CI pipeline MUST fail if new router endpoints lack response_model declarations or validation models use untyped dictionaries. Code review MUST block merge if Pydantic validation is bypassed. Architecture review is required for any exceptions, documented with EXC-ID reference and expiration date.
</enforcement>