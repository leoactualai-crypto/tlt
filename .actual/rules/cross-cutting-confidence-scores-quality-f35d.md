# Enforce Pydantic BaseModel Validation at External Client Boundaries: Confidence Scores Quality

These rules are ALWAYS ACTIVE for all components that accept data from external clients, including HTTP endpoints, event handlers, and inter-service communication boundaries.

### Rules

- **R-CONF-001** MUST: Confidence scores, quality ratings, and other bounded metrics MUST enforce range constraints at the schema level to prevent out-of-range values from entering business logic.

### Verify

```bash
# Discover the project's dependency manifest and identify the schema validation framework
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'setup.py' | head -1

# Inspect the lock file to determine the exact resolved version
find . -name 'poetry.lock' -o -name 'Pipfile.lock' -o -name 'uv.lock' | head -1

# Locate validation schema definitions and verify they inherit from BaseModel with explicit field constraints
grep -r "class.*BaseModel" --include="*.py" | grep -E "(confidence|quality|score)" | head -20

# Identify HTTP endpoint handlers and verify they declare typed request and response models
grep -r "@app\|@router" --include="*.py" -A 5 | grep -E "(confidence|quality|score)" | head -20

# Locate event processing workflows and verify they instantiate validation schemas
grep -r "CloudEventContext\|DiscordContext\|EventContext" --include="*.py" | head -20

# Verify Field constraints with range validation (ge, le, gt, lt)
grep -r "Field.*ge=\|Field.*le=\|Field.*gt=\|Field.*lt=" --include="*.py" | grep -E "(confidence|quality|score)" | head -20
```

**Accept when:**
- All external client ingress points that handle confidence scores, quality ratings, or bounded metrics define validation schemas with explicit type annotations and range constraints (ge, le, gt, lt).
- HTTP endpoint handlers declare typed request and response models with Field constraints that are automatically validated by the web framework.
- Event processing workflows validate incoming payloads against context-specific schemas before state transitions or business logic execution.
- Validation errors are captured with structured error messages identifying the field, constraint violation, and expected format.
- Confidence scores and quality metrics are constrained to valid ranges (e.g., 0.0 to 1.0) at the schema definition level, not in business logic.

<enforcement>
Claude Code MUST NOT skip or defer verification. All new endpoints accepting confidence scores, quality ratings, or bounded metrics from external clients MUST define Pydantic BaseModel schemas with explicit range constraints before merging.
</enforcement>