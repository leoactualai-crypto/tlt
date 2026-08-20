# Adopt Loguru for Logging in MCP Gateway Service: Mcp Gateway Service Use Loguru Logging

These rules are ALWAYS ACTIVE for all modules within the `tlt.mcp_services.gateway` package, including service entry points, resource handlers, request processing modules, and any components requiring operational visibility or debugging output.

### Rules

- **R-LOGURU-001** MUST: The MCP gateway service MUST use Loguru as the logging library implementation for all in-scope modules.
- **R-LOGURU-002** MUST: Import the logger instance at module level using the canonical pattern `from loguru import logger` to ensure consistent access across all functions and classes within the module.
- **R-LOGURU-003** MUST: When configuring log levels from environment variables, implement validation and fallback to default levels to prevent runtime errors from invalid configuration.
- **R-LOGURU-004** SHOULD: For error logging scenarios, use the logger's `exception()` method to automatically capture and format stack traces.
- **R-LOGURU-005** MUST: Do not use Python standard library logging module or alternative logging implementations within gateway service modules.

### Verify

```bash
# Discover the project's dependency manifest and locate the logging library declaration
find . -name "pyproject.toml" -o -name "requirements.txt" -o -name "setup.py" | xargs grep -l "loguru" 2>/dev/null

# Verify that logging imports follow the canonical pattern across gateway service modules
grep -r "from loguru import logger" tlt/mcp_services/gateway/ --include="*.py"

# Verify no stdlib logging imports in gateway service modules
grep -r "import logging" tlt/mcp_services/gateway/ --include="*.py" | grep -v "# noqa" || echo "No stdlib logging imports found (good)"

# Execute the project's test suite for the gateway service
python -m pytest tlt/mcp_services/gateway/ -v

# Verify logging library presence in dependency lock artifact
grep -i "loguru" $(find . -name "*.lock" -o -name "poetry.lock" -o -name "Pipfile.lock" 2>/dev/null | head -1)
```

**Accept when:**
- All gateway service modules import the logger using the canonical import pattern `from loguru import logger`.
- The dependency manifest declares Loguru with appropriate version constraints.
- Service initialization and request handling code produces structured log output without configuration errors.
- No stdlib logging or alternative logging library imports are detected in gateway service modules.
- The gateway service test suite executes without logging-related failures or output pollution.
- Loguru is present in the project's dependency lock artifact.

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for gateway service code review and CI pipeline checks.
</enforcement>