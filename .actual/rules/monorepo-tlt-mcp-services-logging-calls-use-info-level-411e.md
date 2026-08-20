# Adopt Loguru for Logging in MCP Gateway Service: Logging Calls Use Info Level Method

These rules are ALWAYS ACTIVE for all modules within the `tlt.mcp_services.gateway` package, including service entry points, resource handlers, request processing modules, and components requiring operational visibility or debugging output.

### Rules

- **R-LOGURU-001** MUST: Import the logger instance at module level using the canonical pattern `from loguru import logger` to ensure consistent access across all functions and classes within the module.
- **R-LOGURU-002** SHOULD: Logging calls SHOULD use the info-level method for normal operational events, including service startup and configuration confirmation.
- **R-LOGURU-003** MUST: For error logging scenarios, use the logger's `exception()` method to automatically capture and format stack traces.
- **R-LOGURU-004** MUST: When configuring log levels from environment variables, implement validation and fallback to default levels to prevent runtime errors from invalid configuration.
- **R-LOGURU-005** MUST NOT: Use Python standard library logging module or alternative logging implementations within gateway service code; use Loguru exclusively.

### Verify

```bash
# Discover the project's dependency manifest and locate the logging library declaration
find . -name 'pyproject.toml' -o -name 'requirements.txt' -o -name 'setup.py' | xargs grep -l 'loguru' || echo 'Loguru not found in dependency manifest'

# Discover and execute the project's import validation or linting tooling
# to verify that logging imports follow the canonical pattern across gateway service modules
grep -r "from loguru import logger" tlt/mcp_services/gateway/ || echo 'No canonical loguru imports found'

# Verify no stdlib logging imports in gateway service modules
grep -r "import logging" tlt/mcp_services/gateway/ && echo 'WARNING: stdlib logging found' || echo 'No stdlib logging imports detected'

# Discover and execute the project's test suite for the gateway service
# to verify that logging calls do not cause test failures or output pollution
python -m pytest tlt/mcp_services/gateway/ -v --tb=short 2>&1 | head -50
```

**Accept when:**
- All gateway service modules import the logger using the canonical import pattern `from loguru import logger`.
- The dependency manifest declares Loguru with appropriate version constraints.
- Service initialization and request handling code produces structured log output without configuration errors.
- No stdlib logging or alternative logging library imports are present in gateway service modules.
- The project's test suite executes without logging-related failures or output pollution.

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for gateway service logging implementation. Code review and CI pipeline checks MUST enforce compliance before merge.
</enforcement>