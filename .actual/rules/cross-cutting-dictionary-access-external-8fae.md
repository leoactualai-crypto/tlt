# Validate External Client Responses with Pydantic Models and Timeout Controls: Dictionary Access External

These rules are ALWAYS ACTIVE for all files that interact with external HTTP clients, configuration sources, user-provided input, or JSON data loaded from external files or network sources.

### Rules

- **R-EX-001** MUST: All dictionary access to external response data MUST use `.get()` accessor with explicit default values rather than direct key access to handle missing fields safely.
- **R-EX-002** MUST: All HTTP requests using the requests library to external URLs MUST include explicit timeout parameters (recommended: 30s for external APIs, 5s for internal services).
- **R-EX-003** MUST: All external data structures MUST be validated through Pydantic BaseModel subclasses with Field constraints before processing.
- **R-EX-004** SHOULD: Define Pydantic BaseModel subclasses for each external API contract with Field constraints documenting expected ranges, formats, and semantics.
- **R-EX-005** SHOULD: Add loguru logger statements to capture validation failures with structured context (service name, endpoint, error details) for security monitoring.

### Verify

```bash
# Check for requests.get() calls without timeout parameters
grep -r 'requests\.get(' --include='*.py' | grep -v 'timeout=' && echo 'FAIL: Found requests.get without timeout' || echo 'PASS: All requests have timeout'

# Verify Pydantic Field constraints are present
grep -r 'class.*BaseModel' --include='*.py' -A 5 | grep -c 'Field(' && echo 'Pydantic Field constraints found'

# Check for direct dictionary access without .get()
grep -r '\["' --include='*.py' | grep -v '.get(' | grep -v '#' && echo 'WARN: Found direct dict access without .get()' || echo 'PASS: Using safe .get() accessors'
```

**Accept when:**
- All HTTP requests to external clients include explicit timeout parameters verified by grep pattern matching
- All external data structures are validated through Pydantic BaseModel subclasses with Field constraints before processing
- Code review confirms `.get()` accessor usage with defaults for all external response dictionary access
- Security testing confirms services handle malformed external responses without crashes or injection vulnerabilities

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks MUST block commits containing requests.get() without timeout parameters. Code review MUST require Pydantic validation for all new external client integrations. Static analysis tools MUST enforce type safety on external data models. Integration tests MUST validate timeout behavior and validation error handling.
</enforcement>