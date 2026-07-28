# Validate External Client Responses with Pydantic Models and Timeout Controls: Numeric Fields External

These rules are ALWAYS ACTIVE for all HTTP client integrations, external API calls, configuration loading from environment variables, user-provided input through adapters, and JSON data from external files or network sources.

### Rules

- **R-EX-001** MUST: Numeric fields from external sources MUST define range constraints using Field(ge=min, le=max) to enforce valid bounds.

### Verify

```bash
# Check that all requests.get() calls include timeout parameters
grep -r 'requests\.get(' --include='*.py' | grep -v 'timeout=' && echo 'FAIL: Found requests.get without timeout' || echo 'PASS: All requests have timeout'

# Verify Pydantic Field constraints are present in BaseModel definitions
grep -r 'class.*BaseModel' --include='*.py' -A 5 | grep -c 'Field(' && echo 'Pydantic Field constraints found'

# Check for unsafe direct dictionary access without .get()
grep -r '\["' --include='*.py' | grep -v '.get(' | grep -v '#' && echo 'WARN: Found direct dict access without .get()' || echo 'PASS: Using safe .get() accessors'
```

**Accept when:**
- All HTTP requests to external clients include explicit timeout parameters (e.g., timeout=30)
- All external data structures are validated through Pydantic BaseModel subclasses with Field constraints before processing
- Code review confirms .get() accessor usage with defaults for all external response dictionary access
- Security testing confirms services handle malformed external responses without crashes or injection vulnerabilities
- Numeric fields in Pydantic models for external data include ge (greater than or equal) and le (less than or equal) constraints

<enforcement>
Claude Code MUST NOT skip or defer verification. All external numeric fields MUST have explicit range constraints before merge.
</enforcement>