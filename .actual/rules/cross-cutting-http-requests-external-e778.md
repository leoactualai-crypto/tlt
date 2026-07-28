# Validate External Client Responses with Pydantic Models and Timeout Controls: Http Requests External

These rules are ALWAYS ACTIVE for all HTTP requests to external clients, external API responses, configuration loaded from environment variables, user-provided input through adapters or endpoints, and JSON data from external files or network sources.

### Rules

- **R-EX-001** MUST: All HTTP requests to external clients MUST specify an explicit timeout parameter (e.g., `requests.get(url, timeout=30)`) to prevent resource exhaustion and hanging connections.
- **R-EX-002** MUST: All external data structures MUST be validated through Pydantic BaseModel subclasses with Field constraints before processing to prevent injection attacks and type confusion.
- **R-EX-003** MUST: All dictionary access to external response data MUST use `.get()` accessor patterns with fallback defaults (e.g., `data.get('key', default_value)`) to prevent KeyError exceptions.
- **R-EX-004** SHOULD: Pydantic Field constraints SHOULD document expected ranges, formats, and semantics (e.g., `Field(ge=0.0, le=1.0, description="score between 0 and 1")`).
- **R-EX-005** SHOULD: Validation failures SHOULD be captured with loguru logger statements including structured context (service name, endpoint, error details) for security monitoring.

### Verify

```bash
# Check for requests.get() calls without timeout parameters
grep -r 'requests\.get(' --include='*.py' | grep -v 'timeout=' && echo 'FAIL: Found requests.get without timeout' || echo 'PASS: All requests have timeout'

# Verify Pydantic BaseModel usage with Field constraints
grep -r 'class.*BaseModel' --include='*.py' -A 5 | grep -c 'Field(' && echo 'Pydantic Field constraints found'

# Check for unsafe direct dictionary access without .get()
grep -r '\["' --include='*.py' | grep -v '.get(' | grep -v '#' && echo 'WARN: Found direct dict access without .get()' || echo 'PASS: Using safe .get() accessors'
```

**Accept when:**
- All HTTP requests to external clients include explicit timeout parameters verified by grep pattern matching
- All external data structures are validated through Pydantic BaseModel subclasses with Field constraints before processing
- Code review confirms `.get()` accessor usage with defaults for all external response dictionary access
- Security testing confirms services handle malformed external responses without crashes or injection vulnerabilities
- Timeout values are documented and aligned with service SLAs (e.g., 30s for external APIs, 5s for internal services)

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All HTTP requests to external clients MUST include timeout parameters, all external data MUST be validated with Pydantic models, and all dictionary access MUST use safe `.get()` patterns. Pre-commit hooks and code review are mandatory enforcement points.
</enforcement>