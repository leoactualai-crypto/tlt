# Enforce Pydantic BaseModel Validation at External Client Boundaries: Validation Schemas Define

These rules are ALWAYS ACTIVE for all components that accept data from external clients, including HTTP endpoints, event handlers, and inter-service communication boundaries.

### Rules

- **R-VAL-001** MAY: Validation schemas MAY define custom validators for complex business rules that cannot be expressed through declarative field constraints alone.

### Verify

```bash
# Discover the project's dependency manifest and identify the schema validation framework
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'setup.py' | head -1

# Inspect the lock file to determine the exact resolved version
find . -name 'poetry.lock' -o -name 'Pipfile.lock' -o -name 'uv.lock' | head -1

# Locate validation schema definitions in the codebase
grep -r "class.*BaseModel" --include="*.py" | grep -E "(schema|model|validation)" | head -20

# Identify HTTP endpoint handlers and verify they declare typed request and response models
grep -r "@app\|@router" --include="*.py" -A 5 | grep -E "def |: " | head -20

# Locate event processing workflows and verify they instantiate validation schemas
grep -r "CloudEventContext\|DiscordContext\|TimerContext\|EventContext" --include="*.py" | head -20

# Verify validation error handling
grep -r "ValidationError\|except.*Error" --include="*.py" | grep -i valid | head -20
```

**Accept when:**
- All external client ingress points define validation schemas with explicit type annotations, required field declarations, and constraint specifications.
- HTTP endpoint handlers declare typed request and response models that are automatically validated by the web framework.
- Event processing workflows validate incoming payloads against context-specific schemas before state transitions or business logic execution.
- Validation errors are captured with structured error messages identifying the field, constraint violation, and expected format.
- Custom validators are defined only for complex business rules that cannot be expressed through declarative field constraints.

<enforcement>
Clause Code MUST NOT skip or defer verification. All external client boundaries MUST implement validation schemas before data enters business logic.
</enforcement>