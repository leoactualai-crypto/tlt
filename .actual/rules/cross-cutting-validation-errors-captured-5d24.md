# Enforce Pydantic BaseModel Validation at External Client Boundaries: Validation Errors Captured

These rules are ALWAYS ACTIVE for all components that accept data from external clients, including HTTP endpoints, event handlers, and inter-service communication boundaries.

### Rules

- **R-VAL-001** SHOULD: Validation errors SHOULD be captured with structured error messages that identify the specific field, constraint violation, and expected format.

### Verify

```bash
# Discover the project's dependency manifest and identify the schema validation framework
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'setup.py' | head -1

# Inspect the lock file to determine the exact resolved version
find . -name 'poetry.lock' -o -name 'Pipfile.lock' -o -name 'uv.lock' | head -1

# Locate validation schema definitions in the codebase
grep -r "class.*BaseModel" --include="*.py" | grep -E "(models|schemas)" | head -20

# Identify HTTP endpoint handlers and verify they declare typed request and response models
grep -r "@app\|@router" --include="*.py" -A 5 | grep -E "def |response_model|Request|Response" | head -20

# Locate event processing workflows and verify they instantiate validation schemas
grep -r "CloudEventContext\|DiscordContext\|TimerContext\|EventContext" --include="*.py" | head -20

# Verify validation error handling in event processors
grep -r "ValidationError\|except.*Error" --include="*.py" -B 2 -A 2 | head -30
```

**Accept when:**
- All external client ingress points define validation schemas with explicit type annotations, required field declarations, and constraint specifications.
- HTTP endpoint handlers declare typed request and response models that are automatically validated by the web framework.
- Event processing workflows validate incoming payloads against context-specific schemas before state transitions or business logic execution.
- Validation errors are captured with structured error messages identifying the field, constraint violation, and expected format.
- Schema definitions inherit from the framework's base class with explicit field constraints (e.g., `ge=0.0`, `le=1.0` for numeric ranges).

<enforcement>
Clause Code MUST NOT skip or defer verification. All new external client endpoints MUST define validation schemas with explicit constraints before merging. Static analysis tooling MUST detect unvalidated external data flows and escalate to errors in CI pipeline. Integration tests MUST verify validation errors are returned with structured messages for invalid payloads.
</enforcement>