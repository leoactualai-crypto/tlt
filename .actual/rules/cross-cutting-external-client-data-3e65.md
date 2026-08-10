# Enforce Pydantic BaseModel Validation at External Client Boundaries: External Client Data

These rules are ALWAYS ACTIVE for all components that accept data from external clients, including HTTP endpoints, event handlers, and inter-service communication boundaries.

### Rules

- **R-EX-001** MUST: All external client data entering service boundaries MUST be validated through a schema validation framework that enforces type constraints, required fields, and value ranges before business logic execution.

### Verify

```bash
# Discover the project's dependency manifest and identify the schema validation framework
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'poetry.lock' -o -name 'Pipfile.lock' | head -1

# Inspect lock file to determine exact resolved version of validation framework
grep -A 5 'pydantic' $(find . -name '*.lock' -o -name 'pyproject.toml' | head -1)

# Locate validation schema definitions and verify they inherit from BaseModel
grep -r 'class.*BaseModel' --include='*.py' | grep -E '(Context|Create|Output|Response)'

# Identify HTTP endpoint handlers and verify typed request/response models
grep -r '@.*route\|@.*post\|@.*get\|@.*delete' --include='*.py' -A 5 | grep -E '(def |: )'

# Locate event processing workflows and verify schema instantiation
grep -r 'CloudEventContext\|DiscordContext\|TimerContext\|EventContext' --include='*.py' -B 2 -A 2

# Verify validation error handling in event processors
grep -r 'ValidationError\|except.*Error' --include='*.py' | grep -E '(event|handler|processor)'
```

**Accept when:**
- All external client ingress points define validation schemas with explicit type annotations, required field declarations, and constraint specifications.
- HTTP endpoint handlers declare typed request and response models that are automatically validated by the web framework.
- Event processing workflows validate incoming payloads against context-specific schemas before state transitions or business logic execution.
- Validation errors are captured with structured error messages identifying the field, constraint violation, and expected format.
- Schema definitions include descriptive field documentation to generate self-documenting API contracts.

<enforcement>
Claude Code MUST NOT skip or defer verification. All new external client endpoints MUST define validation schemas with explicit constraints before merging. Static analysis tooling MUST detect unvalidated external data flows and escalate to errors in CI pipeline. Integration tests MUST verify validation errors are returned with structured messages for invalid payloads.
</enforcement>