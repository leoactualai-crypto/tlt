# Adopt Loguru for Logging in MCP Gateway Service: Before Implementing Logging Calls Developers Discover

These rules are ALWAYS ACTIVE for all modules within the `tlt.mcp_services.gateway` package, service entry points, resource handlers, request processing modules, and any components requiring operational visibility or debugging output.

### Rules

- **R-LOGURU-001** MUST: Before implementing logging calls, developers MUST discover the project's dependency lock artifact and verify the exact resolved version of the logging library.
- **R-LOGURU-002** MUST: Import the logger instance at module level using the canonical pattern `from loguru import logger` to ensure consistent access across all functions and classes within the module.
- **R-LOGURU-003** MUST: When configuring log levels from environment variables, implement validation and fallback to default levels to prevent runtime errors from invalid configuration.
- **R-LOGURU-004** SHOULD: For error logging scenarios, use the logger's `exception()` method to automatically capture and format stack traces.
- **R-LOGURU-005** MUST: All gateway service modules use the adopted logging library; stdlib logging or alternative logging implementations are not permitted within the gateway service boundary.

### Verify

```bash
# 1. Discover the project's dependency manifest and locate the logging library declaration
find . -name "pyproject.toml" -o -name "requirements.txt" -o -name "Pipfile" | head -1

# 2. Inspect the lock artifact to determine the exact resolved version
find . -name "*.lock" -o -name "poetry.lock" -o -name "Pipfile.lock" | head -1

# 3. Verify logging imports follow the canonical pattern across gateway service modules
grep -r "from loguru import logger" tlt/mcp_services/gateway/ || echo "No canonical imports found"

# 4. Detect any non-compliant logging imports (stdlib logging)
grep -r "import logging" tlt/mcp_services/gateway/ && echo "WARNING: stdlib logging detected" || echo "No stdlib logging imports"

# 5. Execute the project's test suite for the gateway service
python -m pytest tlt/mcp_services/gateway/ -v

# 6. Verify logging library presence in dependency lock artifact
grep -i "loguru" $(find . -name "*.lock" | head -1)
```

**Accept when:**
- All gateway service modules import the logger using the canonical import pattern `from loguru import logger`.
- The dependency manifest declares the logging library with appropriate version constraints.
- The lock artifact contains the exact resolved version of the logging library.
- Service initialization and request handling code produces structured log output without configuration errors.
- The test suite for the gateway service passes without logging-related failures or output pollution.
- No stdlib logging imports are detected in gateway service modules.

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory before logging calls are implemented. Violations block code review and CI pipeline progression.
</enforcement>