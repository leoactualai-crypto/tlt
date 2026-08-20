# Adopt Loguru for Logging in MCP Gateway Service: Services Configure Log Levels Through Environment

These rules are ALWAYS ACTIVE for all modules within the `tlt.mcp_services.gateway` package, service entry points, resource handlers, request processing modules, and components requiring operational visibility or debugging output.

### Rules

- **R-LOGURU-001** MUST: Import the logger instance at module level using the canonical pattern `from loguru import logger` to ensure consistent access across all functions and classes within the module.
- **R-LOGURU-002** MUST: Use Loguru for all logging operations in gateway service modules instead of Python's standard library logging module.
- **R-LOGURU-003** SHOULD: Configure log levels through environment variables discovered from the project's configuration management patterns.
- **R-LOGURU-004** SHOULD: Use the logger's `exception()` method to automatically capture and format stack traces in error logging scenarios.
- **R-LOGURU-005** SHOULD: Implement validation and fallback to default log levels when configuring from environment variables to prevent runtime errors from invalid configuration.
- **R-LOGURU-006** MAY: Services MAY configure log levels through environment variables discovered from the project's configuration management patterns.

### Verify

```bash
# Discover the project's dependency manifest and locate the logging library declaration
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'setup.py' | xargs grep -l 'loguru' || echo "Loguru not found in dependency manifest"

# Discover and execute the project's import validation or linting tooling
# (Assumes ruff, pylint, or similar is configured in the project)
grep -r "from loguru import logger" tlt/mcp_services/gateway/ || echo "No canonical loguru imports found"

# Verify no stdlib logging imports in gateway service modules
grep -r "import logging" tlt/mcp_services/gateway/ && echo "WARNING: stdlib logging found" || echo "No stdlib logging imports detected"

# Discover and execute the project's test suite for the gateway service
# (Assumes pytest or similar is configured)
python -m pytest tlt/mcp_services/gateway/ -v 2>&1 | grep -E '(PASSED|FAILED|ERROR)' || echo "Test execution discovery required"
```

**Accept when:**
- All gateway service modules import the logger using the canonical import pattern `from loguru import logger`.
- The dependency manifest declares Loguru with appropriate version constraints.
- Service initialization and request handling code produces structured log output without configuration errors.
- No stdlib logging or alternative logging library imports are present in gateway service modules.
- The gateway service test suite executes without logging-related failures or output pollution.

<enforcement>
Claude Code MUST NOT skip or defer verification. All R-LOGURU rules marked MUST are mandatory for code review acceptance. Code review MUST reject pull requests introducing stdlib logging or alternative logging libraries in gateway service code. Automated linting MUST block CI pipeline progression when non-compliant logging imports are detected.
</enforcement>