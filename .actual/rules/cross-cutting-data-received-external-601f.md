# Validate External Client Responses with Pydantic Models and Timeout Controls: Data Received External

These rules are ALWAYS ACTIVE for all code that receives data from external HTTP clients, APIs, webhooks, configuration sources, or user input through network boundaries.

### Rules

- **R-EX-001** MUST: All data received from external clients MUST be validated using Pydantic BaseModel subclasses with explicit Field constraints before processing.
- **R-EX-002** MUST: All HTTP requests using the requests library to external URLs MUST include explicit timeout parameters (recommended: 30s for external APIs, 5s for internal services).
- **R-EX-003** MUST: All dictionary access to external response data MUST use .get() accessor patterns with fallback defaults instead of direct bracket notation.
- **R-EX-004** SHOULD: Validation failures and timeout events SHOULD be logged with loguru logger statements capturing structured context (service name, endpoint, error details) for security monitoring.

### Verify

```bash
# Check for requests.get() calls without timeout parameters
grep -r 'requests\.get(' --include='*.py' | grep -v 'timeout=' && echo 'FAIL: Found requests.get without timeout' || echo 'PASS: All requests have timeout'

# Verify Pydantic BaseModel usage with Field constraints
grep -r 'class.*BaseModel' --include='*.py' -A 5 | grep -c 'Field(' && echo 'Pydantic Field constraints found'

# Check for unsafe direct dictionary access patterns
grep -r '\["' --include='*.py' | grep -v '.get(' | grep -v '#' && echo 'WARN: Found direct dict access without .get()' || echo 'PASS: Using safe .get() accessors'
```

**Accept when:**
- All HTTP requests to external clients include explicit timeout parameters verified by grep pattern matching
- All external data structures are validated through Pydantic BaseModel subclasses with Field constraints before processing
- Code review confirms .get() accessor usage with defaults for all external response dictionary access
- Security testing confirms services handle malformed external responses without crashes or injection vulnerabilities
- Validation failures are logged with structured context for security monitoring and incident response

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All external data boundaries MUST be protected with Pydantic validation, timeout controls, and safe accessor patterns before code is approved for merge.
</enforcement>