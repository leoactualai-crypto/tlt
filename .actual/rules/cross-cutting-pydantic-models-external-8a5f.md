# Validate External Client Responses with Pydantic Models and Timeout Controls: Pydantic Models External

These rules are ALWAYS ACTIVE for all HTTP requests to external clients, external API responses, configuration loaded from environment variables, user-provided input through adapters, and JSON data from external files or network sources.

### Rules

- **R-EX-001** MUST: Pydantic models for external data MUST include description fields documenting expected semantics and constraints.
- **R-EX-002** MUST: All HTTP requests using the requests library to external URLs MUST include explicit timeout parameters.
- **R-EX-003** MUST: All external data structures MUST be validated through Pydantic BaseModel subclasses with Field constraints before processing.
- **R-EX-004** MUST: All dictionary access to external response data MUST use .get() accessor patterns with fallback defaults instead of direct bracket notation.
- **R-EX-005** SHOULD: Timeout values SHOULD follow standards: 30 seconds for external APIs, 5 seconds for internal services.
- **R-EX-006** SHOULD: Validation failures SHOULD be captured with loguru logger statements including structured context (service name, endpoint, error details) for security monitoring.

### Verify

```bash
# Check for requests.get() calls without timeout parameters
grep -r 'requests\.get(' --include='*.py' | grep -v 'timeout=' && echo 'FAIL: Found requests.get without timeout' || echo 'PASS: All requests have timeout'

# Verify Pydantic Field constraints are present
grep -r 'class.*BaseModel' --include='*.py' -A 5 | grep -c 'Field(' && echo 'Pydantic Field constraints found'

# Check for unsafe direct dictionary access without .get()
grep -r '\["' --include='*.py' | grep -v '.get(' | grep -v '#' && echo 'WARN: Found direct dict access without .get()' || echo 'PASS: Using safe .get() accessors'

# Verify Pydantic models have description fields
grep -r 'Field(' --include='*.py' -A 1 | grep -c 'description=' && echo 'Description fields found in Pydantic models'
```

**Accept when:**
- All HTTP requests to external clients include explicit timeout parameters verified by grep pattern matching
- All external data structures are validated through Pydantic BaseModel subclasses with Field constraints before processing
- Code review confirms .get() accessor usage with defaults for all external response dictionary access
- All Pydantic models for external data include description fields documenting expected semantics and constraints
- Security testing confirms services handle malformed external responses without crashes or injection vulnerabilities
- Timeout values follow documented standards (30s for external APIs, 5s for internal services) or are justified with performance profiling

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code touching external client boundaries. Pre-commit hooks and code review processes enforce compliance before merge.
</enforcement>