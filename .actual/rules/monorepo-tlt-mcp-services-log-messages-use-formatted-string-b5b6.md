# Adopt Loguru for Logging in MCP Gateway Service: Log Messages Use Formatted String Literals

These rules are ALWAYS ACTIVE for all modules within the `tlt.mcp_services.gateway` package, including service entry points, resource handlers, request processing modules, and components requiring operational visibility or debugging output.

### Rules

- **R-LOGURU-001** MUST: Import the logger instance at module level using the canonical pattern `from loguru import logger` to ensure consistent access across all functions and classes within the module.
- **R-LOGURU-002** SHOULD: Log messages SHOULD use formatted string literals to embed runtime values and context.
- **R-LOGURU-003** MUST: Use the logger's `exception()` method to automatically capture and format stack traces in error logging scenarios.
- **R-LOGURU-004** MUST: When configuring log levels from environment variables, implement validation and fallback to default levels to prevent runtime errors from invalid configuration.
- **R-LOGURU-005** MUST NOT: Use Python standard library `logging` module or alternative logging implementations within gateway service code.

### Verify

```bash
# Discover the project's dependency manifest and locate the logging library declaration
find . -name "pyproject.toml" -o -name "requirements.txt" -o -name "setup.py" | xargs grep -l "loguru"

# Verify that logging imports follow the canonical pattern across gateway service modules
grep -r "from loguru import logger" tlt/mcp_services/gateway/ --include="*.py"

# Verify no stdlib logging imports in gateway service modules
grep -r "import logging" tlt/mcp_services/gateway/ --include="*.py" | grep -v "# noqa" || echo "No stdlib logging imports found"

# Execute the project's test suite for the gateway service
python -m pytest tlt/mcp_services/gateway/ -v

# Verify logging library presence in dependency lock artifact
grep -i "loguru" poetry.lock || grep -i "loguru" requirements.lock || echo "Check lock file for loguru"
```

**Accept when:**
- All gateway service modules import the logger using the canonical import pattern `from loguru import logger`.
- The dependency manifest declares loguru with appropriate version constraints.
- Service initialization and request handling code produces structured log output without configuration errors.
- No stdlib logging imports are detected in gateway service modules (excluding third-party library integrations).
- The project's test suite executes without logging-related failures or output pollution.
- Loguru is present in the project's dependency lock artifact with a resolved version.

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for gateway service logging implementation.
</enforcement>