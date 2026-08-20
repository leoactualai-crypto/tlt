# Adopt Loguru for Logging in MCP Gateway Service: Modules Requiring Logging Import Logger Instance

These rules are ALWAYS ACTIVE for all modules within the `tlt.mcp_services.gateway` package, service entry points that initialize the gateway application, resource handlers, request processing modules, and components that require operational visibility or debugging output.

### Rules

- **R-LOGURU-001** MUST: Modules requiring logging MUST import the logger instance using the canonical import pattern `from loguru import logger`.
- **R-LOGURU-002** MUST: Import the logger instance at module level to ensure consistent access across all functions and classes within the module.
- **R-LOGURU-003** SHOULD: When configuring log levels from environment variables, implement validation and fallback to default levels to prevent runtime errors from invalid configuration.
- **R-LOGURU-004** SHOULD: For error logging scenarios, use the logger's `exception()` method to automatically capture and format stack traces.

### Verify

```bash
# Discover the project's dependency manifest and locate the logging library declaration
find . -name "pyproject.toml" -o -name "requirements.txt" -o -name "setup.py" | head -1 | xargs grep -i loguru

# Discover and execute the project's import validation or linting tooling
# (e.g., ruff, pylint, flake8) to verify logging imports follow the canonical pattern
ruff check --select F401 tlt/mcp_services/gateway/ 2>/dev/null || echo "(linting tool not found or not configured)"

# Discover and execute the project's test suite for the gateway service
pytest tlt/mcp_services/gateway/ -v 2>/dev/null || echo "(test suite not found or not configured)"

# Verify canonical import pattern across gateway modules
grep -r "from loguru import logger" tlt/mcp_services/gateway/ || echo "(no loguru imports found)"

# Verify no stdlib logging imports in gateway service modules
grep -r "import logging" tlt/mcp_services/gateway/ | grep -v "# noqa" || echo "(no stdlib logging imports found)"
```

**Accept when:**
- All gateway service modules import the logger using the canonical import pattern `from loguru import logger`.
- The dependency manifest declares loguru with appropriate version constraints.
- Service initialization and request handling code produces structured log output without configuration errors.
- No stdlib logging or alternative logging library imports are detected in gateway service modules.
- The project's test suite executes without logging-related failures or output pollution.

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code within the specified scope.
</enforcement>