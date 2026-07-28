# Validate External Client Responses with Pydantic Models and Timeout Controls: External Client Error

These rules are ALWAYS ACTIVE for all HTTP client integrations, external API calls, configuration loading from environment variables, user input from Discord adapters or API endpoints, and JSON data from external files or network sources.

### Rules

- **R-EX-001** MUST: All HTTP requests using the requests library to external URLs SHALL include explicit timeout parameters (recommended: 30s for external APIs, 5s for internal services).
- **R-EX-002** MUST: All external data structures received from HTTP responses, APIs, webhooks, or configuration sources SHALL be validated through Pydantic BaseModel subclasses with Field constraints before processing.
- **R-EX-003** SHOULD: External client error responses SHOULD be checked explicitly (e.g., `result.get('error')`) before processing success paths.
- **R-EX-004** MUST: All dictionary access to external response data SHALL use `.get()` accessor patterns with explicit defaults (e.g., `data.get('key', default_value)`) to prevent KeyError exceptions.
- **R-EX-005** SHOULD: Validation failures and external client errors SHOULD be logged with structured context (service name, endpoint, error details) using loguru logger for security monitoring.

### Verify

```bash
# Verify all requests.get() calls include timeout parameters
grep -r 'requests\.get(' --include='*.py' | grep -v 'timeout=' && echo 'FAIL: Found requests.get without timeout' || echo 'PASS: All requests have timeout'

# Verify Pydantic Field constraints are used for external data validation
grep -r 'class.*BaseModel' --include='*.py' -A 5 | grep -c 'Field(' && echo 'Pydantic Field constraints found'

# Verify safe .get() accessor usage for external response data
grep -r '\["' --include='*.py' | grep -v '.get(' | grep -v '#' && echo 'WARN: Found direct dict access without .get()' || echo 'PASS: Using safe .get() accessors'
```

**Accept when:**
- All HTTP requests to external clients include explicit timeout parameters verified by grep pattern matching
- All external data structures are validated through Pydantic BaseModel subclasses with Field constraints before processing
- Code review confirms `.get()` accessor usage with defaults for all external response dictionary access
- Security testing confirms services handle malformed external responses without crashes or injection vulnerabilities
- External client error responses are explicitly checked before success path processing
- Validation failures are logged with structured context for security monitoring

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All external client integrations MUST satisfy R-EX-001, R-EX-002, and R-EX-004 before code review approval. Violations block commit via pre-commit hooks and CI pipeline security scanning.
</enforcement>