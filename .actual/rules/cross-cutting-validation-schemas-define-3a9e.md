# Enforce Pydantic BaseModel Validation at External Client Boundaries: Validation Schemas Define

These rules are ALWAYS ACTIVE for all components that accept data from external clients, including HTTP endpoints, event handlers, and inter-service communication boundaries.

### Rules

- **R-VAL-001** MUST: Validation schemas MUST define explicit field constraints including type annotations, nullability, default values, and range constraints for numeric fields.
- **R-VAL-002** MUST: Define validation schemas as separate classes that inherit from the schema validation framework's base class, using field constraint declarations to enforce type, nullability, defaults, and range constraints.
- **R-VAL-003** MUST: For HTTP endpoints, declare request body and response models using the web framework's type annotation system to enable automatic validation and serialization.
- **R-VAL-004** MUST: For event processing workflows, validate incoming event payloads by instantiating the appropriate context schema class and catching validation exceptions to handle malformed events.
- **R-VAL-005** SHOULD: Include descriptive field documentation in schema definitions to generate self-documenting API contracts and improve external client integration experience.

### Verify

```bash
# Discover the project's dependency manifest and identify the schema validation framework
grep -r "pydantic" . --include="*.txt" --include="*.toml" --include="*.cfg" | head -5

# Inspect the lock file to determine the exact resolved version
find . -name "*.lock" -o -name "poetry.lock" -o -name "requirements.lock" | xargs cat | grep -i pydantic

# Locate validation schema definitions in the codebase
grep -r "class.*BaseModel" . --include="*.py" | grep -v test | grep -v __pycache__

# Verify HTTP endpoint handlers declare typed request and response models
grep -r "@app\|@router" . --include="*.py" -A 3 | grep -E "def |: .*Model"

# Locate event processing workflows and verify schema instantiation
grep -r "CloudEventContext\|DiscordContext\|TimerContext\|EventContext" . --include="*.py" | grep -v test

# Verify validation error handling
grep -r "ValidationError\|except.*Error" . --include="*.py" | grep -v test
```

**Accept when:**
- All external client ingress points define validation schemas with explicit type annotations, required field declarations, and constraint specifications.
- HTTP endpoint handlers declare typed request and response models that are automatically validated by the web framework.
- Event processing workflows validate incoming payloads against context-specific schemas before state transitions or business logic execution.
- Validation errors are captured with structured error messages identifying the field, constraint violation, and expected format.
- Validation schemas inherit from the framework's base class (e.g., Pydantic BaseModel) with Field constraints for ranges, nullability, and defaults.
- Descriptive field documentation is present in schema definitions to support API contract clarity.

<enforcement>
Clause Code MUST NOT skip or defer verification. All external client boundaries MUST implement validation schemas with explicit constraints before data enters business logic. Violations detected in code review or static analysis MUST be rejected or escalated to engineering lead with security review.
</enforcement>