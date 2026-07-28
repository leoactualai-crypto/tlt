# Validate External Client Responses with Pydantic Models and Timeout Controls: Environment Variable Configuration

These rules are ALWAYS ACTIVE for all HTTP client integrations, external API calls, environment variable configuration loading, and user-provided input validation across the codebase.

### Rules

- **R-ENV-001** SHOULD: Environment variable configuration SHOULD provide fallback defaults using `os.getenv(key, default)` to ensure graceful degradation.
- **R-EXT-001** MUST: All HTTP requests using the requests library to external URLs MUST include explicit timeout parameters (recommended: 30s for external APIs, 5s for internal services).
- **R-EXT-002** MUST: All data received from external APIs, webhooks, or HTTP endpoints MUST be validated through Pydantic BaseModel subclasses with Field constraints before processing.
- **R-EXT-003** MUST: All dictionary access to external response data MUST use `.get()` accessor patterns with fallback defaults instead of direct bracket notation.
- **R-EXT-004** SHOULD: Validation failures on external data SHOULD be captured with loguru logger statements including structured context (service name, endpoint, error details) for security monitoring.

### Verify

```bash
# Check for requests.get() calls without timeout parameters
grep -r 'requests\.get(' --include='*.py' | grep -v 'timeout=' && echo 'FAIL: Found requests.get without timeout' || echo 'PASS: All requests have timeout'

# Verify Pydantic Field constraints are used
grep -r 'class.*BaseModel' --include='*.py' -A 5 | grep -c 'Field(' && echo 'Pydantic Field constraints found'

# Check for unsafe direct dictionary access without .get()
grep -r '\["' --include='*.py' | grep -v '.get(' | grep -v '#' && echo 'WARN: Found direct dict access without .get()' || echo 'PASS: Using safe .get() accessors'

# Verify os.getenv usage with defaults
grep -r 'os\.getenv' --include='*.py' | grep -v 'os\.getenv.*,' && echo 'WARN: Found os.getenv without default' || echo 'PASS: All os.getenv calls have defaults'
```

**Accept when:**
- All HTTP requests to external clients include explicit timeout parameters verified by grep pattern matching
- All external data structures are validated through Pydantic BaseModel subclasses with Field constraints before processing
- Code review confirms `.get()` accessor usage with defaults for all external response dictionary access
- All environment variable configuration uses `os.getenv(key, default)` pattern with fallback defaults
- Security testing confirms services handle malformed external responses without crashes or injection vulnerabilities
- Validation failures are logged with structured context for security monitoring

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. Pre-commit hooks MUST block commits containing `requests.get()` without timeout parameters. Code review MUST require Pydantic validation for all new external client integrations. Static analysis tools (mypy with Pydantic plugin) MUST enforce type safety on external data models. Runtime monitoring MUST alert on validation failure rate spikes.
</enforcement>