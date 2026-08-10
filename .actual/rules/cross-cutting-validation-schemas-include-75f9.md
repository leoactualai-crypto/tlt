# Enforce Pydantic BaseModel Validation at External Client Boundaries: Validation Schemas Include

These rules are ALWAYS ACTIVE for all components that accept data from external clients, including HTTP endpoints, event handlers, and inter-service communication boundaries.

### Rules

- **R-VAL-001** SHOULD: Validation schemas SHOULD include descriptive field documentation to serve as self-documenting API contracts for external clients.

### Verify

```bash
# Discover the project's dependency manifest and identify the schema validation framework
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'setup.py' | head -1

# Inspect lock file to determine exact resolved version
find . -name 'poetry.lock' -o -name 'Pipfile.lock' -o -name 'uv.lock' | head -1

# Locate validation schema definitions and verify they inherit from BaseModel
grep -r "class.*BaseModel" --include="*.py" | grep -E "(schema|model|validation)"

# Identify HTTP endpoint handlers and verify typed request/response models
grep -r "@app\.(post|get|put|delete|patch)" --include="*.py" -A 5 | grep -E "(response_model|Request|Response)"

# Locate event processing workflows and verify schema instantiation
grep -r "CloudEventContext\|DiscordContext\|TimerContext\|EventContext" --include="*.py" -B 2 -A 2

# Verify validation errors are captured with structured messages
grep -r "ValidationError\|pydantic" --include="*.py" | grep -E "(except|raise|catch)"
```

**Accept when:**
- All external client ingress points define validation schemas with explicit type annotations, required field declarations, and constraint specifications.
- HTTP endpoint handlers declare typed request and response models that are automatically validated by the web framework.
- Event processing workflows validate incoming payloads against context-specific schemas before state transitions or business logic execution.
- Validation errors are captured with structured error messages identifying the field, constraint violation, and expected format.
- Validation schema classes include docstrings or Field descriptions documenting the purpose and constraints of each field.

<enforcement>
Clause Code MUST NOT skip or defer verification. All external client boundaries MUST implement validation schemas with documented fields before merging to main branch.
</enforcement>