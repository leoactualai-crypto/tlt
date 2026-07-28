# Validate External Client Responses with Pydantic Models and Timeout Controls: Services Log Validation

These rules are ALWAYS ACTIVE for all files matching the configured scope: HTTP requests using the requests library to external URLs, all data received from external APIs/webhooks/HTTP endpoints, all configuration loaded from environment variables, all user-provided input through Discord adapters or API endpoints, and all JSON data loaded from external files or network sources.

### Rules

- **R-EXTVAL-001** SHOULD: Services SHOULD log validation failures from external clients using structured logging (loguru logger) for security monitoring.

### Verify

```bash
# Verify all requests.get() calls include timeout parameters
grep -r 'requests\.get(' --include='*.py' | grep -v 'timeout=' && echo 'FAIL: Found requests.get without timeout' || echo 'PASS: All requests have timeout'

# Verify Pydantic Field constraints are used
grep -r 'class.*BaseModel' --include='*.py' -A 5 | grep -c 'Field(' && echo 'Pydantic Field constraints found'

# Verify safe dictionary access patterns
grep -r '\["' --include='*.py' | grep -v '.get(' | grep -v '#' && echo 'WARN: Found direct dict access without .get()' || echo 'PASS: Using safe .get() accessors'
```

**Accept when:**
- All HTTP requests to external clients include explicit timeout parameters verified by grep pattern matching
- All external data structures are validated through Pydantic BaseModel subclasses with Field constraints before processing
- Code review confirms .get() accessor usage with defaults for all external response dictionary access
- Security testing confirms services handle malformed external responses without crashes or injection vulnerabilities
- Validation failures are logged with structured context (service name, endpoint, error details) using loguru logger

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks block commits containing requests.get() without timeout parameters. Code review process requires remediation of direct dictionary access patterns before merge approval. Security scanning in CI pipeline flags missing Pydantic validation on external endpoints as high-severity findings. Runtime monitoring alerts on validation failure rate spikes indicating potential attack or schema drift.
</enforcement>