# Adopt Pydantic BaseModel for Internal API Domain Validation: Confidence Scores Ratings

These rules are ALWAYS ACTIVE for all internal API implementations requiring domain validation, including FastAPI endpoint models, agent reasoning structures, MCP service schemas, CloudEvent models, and monitoring response structures.

### Rules

- **R-CONF-001** SHOULD: Confidence scores, ratings, and normalized metrics SHOULD use numeric range constraints to enforce valid bounds at validation time.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact
# Resolve the validation framework's exact installed version
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'poetry.lock' -o -name 'Pipfile.lock' | head -5

# 2. Locate the project's test suite discovery mechanism
# Execute validation-related tests
python -m pytest tests/ -k validation -v 2>/dev/null || echo "Test discovery failed"

# 3. Discover the project's static type checking configuration
# Execute the type checker against all modules containing API model definitions
python -m mypy --config-file pyproject.toml . 2>/dev/null || echo "Type checking not configured"

# 4. Search for all classes inheriting from BaseModel
# Verify each includes field constraint declarations and type annotations
grep -r "class.*BaseModel" --include="*.py" | grep -E "(agent|service|endpoint|schema|model)" | head -20

# 5. Verify field constraints on confidence/rating fields
grep -r "Field.*ge=\|Field.*le=\|Field.*gt=\|Field.*lt=" --include="*.py" | grep -iE "(confidence|rating|score|metric)" | head -20
```

**Accept when:**
- All internal API request and response models inherit from Pydantic BaseModel with explicit field type annotations and constraint declarations
- Static type checking passes for all modules containing domain validation models without type errors
- Validation tests demonstrate that field constraints are enforced at runtime and produce structured error messages for invalid inputs
- Confidence scores, ratings, and normalized metrics use numeric range constraints (ge, le, gt, lt) in Field definitions
- All nested structures maintain validation guarantees through typed model references rather than generic dictionaries

<enforcement>
Claude Code MUST NOT skip or defer verification. All new API models must be reviewed against these rules before merge. Type checking and validation tests must pass in CI.
</enforcement>