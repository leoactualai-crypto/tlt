# Enforce Pydantic BaseModel Validation at External Client Boundaries: Http Endpoint Handlers

These rules are ALWAYS ACTIVE for all HTTP endpoint handlers, CloudEvent processors, Discord adapter event receivers, MCP service input processors, agent task submission interfaces, batch processing endpoints, and health check endpoints that accept external client data.

### Rules

- **R-ECBF-001** MUST: HTTP endpoint handlers MUST declare request and response models as typed schema objects that are automatically validated by the web framework.
- **R-ECBF-002** MUST: All external client ingress points (HTTP endpoints, CloudEvent handlers, Discord adapters, MCP processors, agent task interfaces) MUST define validation schemas with explicit type annotations, required field declarations, and constraint specifications.
- **R-ECBF-003** MUST: Event processing workflows MUST validate incoming event payloads by instantiating the appropriate context schema class before passing data to business logic or state management.
- **R-ECBF-004** MUST: Validation schemas MUST inherit from the schema validation framework's base class and use field constraint declarations to enforce type, nullability, defaults, and range constraints.
- **R-ECBF-005** MUST: Validation errors MUST be captured with structured error messages identifying the field, constraint violation, and expected format.
- **R-ECBF-006** SHOULD: Include descriptive field documentation in schema definitions to generate self-documenting API contracts and improve external client integration experience.

### Verify

```bash
# Discover the project's dependency manifest and identify the schema validation framework
grep -r "pydantic" . --include="*.txt" --include="*.toml" --include="*.cfg" --include="*.lock" | head -5

# Inspect the lock file to determine the exact resolved version
find . -name "*.lock" -o -name "requirements*.txt" | xargs grep -i pydantic

# Locate validation schema definitions and verify they inherit from BaseModel
grep -r "class.*BaseModel" . --include="*.py" | grep -v test | grep -v __pycache__

# Identify HTTP endpoint handlers and verify they declare typed request/response models
grep -r "@app\|@router\|@post\|@get\|@put\|@delete" . --include="*.py" -A 3 | grep -E "def |: .*Model"

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
- Validation schemas inherit from the framework's base class (e.g., Pydantic BaseModel) with explicit field constraints.
- New external client endpoints include schema validation definitions in code review.

<enforcement>
Claude Code MUST NOT skip or defer verification. All new HTTP endpoints, event handlers, and external client ingress points MUST include validation schemas before merging. Static analysis tooling MUST detect unvalidated external data flows and escalate to errors in CI pipeline. Post-deployment monitoring MUST alert on unexpected validation error rates.
</enforcement>