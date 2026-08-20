# Adopt Standardized Core Library Stack for Python MCP Services: Configuration Values Retrieved Getenv Appropriate Default

These rules are ALWAYS ACTIVE for all Python service modules in the MCP services layer (monorepo/tlt/mcp_services/*) that manage state, interact with external systems, or require observability and environment-based configuration.

### Rules

- **R-STDLIB-001** MUST: Configuration values MUST be retrieved using `os.getenv` with appropriate default values for local development.
- **R-STDLIB-002** MUST: All MCP service modules MUST import the standardized library stack in this order: stdlib imports first (os, typing, datetime, uuid, json), then third-party imports (loguru).
- **R-STDLIB-003** MUST: Service modules MUST NOT import Python standard library logging module; use loguru for all structured logging.
- **R-STDLIB-004** MUST: Function signatures in service modules MUST use type annotations from the typing module (typing.Optional, typing.List, typing.Dict) for nullable and collection types.
- **R-STDLIB-005** MUST: Timestamps MUST be timezone-aware UTC timestamps using `datetime.now(timezone.utc)`.
- **R-STDLIB-006** MUST: Unique identifiers MUST be generated using UUID v4.
- **R-STDLIB-007** SHOULD: Loguru logger SHOULD be configured at service initialization to establish consistent log formatting, levels, and output destinations.

### Verify

```bash
# Discover the project's dependency manifest and verify loguru is declared as a dependency for the MCP services package
grep -r "loguru" "$(find . -name 'pyproject.toml' -o -name 'requirements*.txt' -o -name 'setup.py' | head -1)"

# Discover the project's static analysis configuration and run the type checker against service modules
find . -path "*/tlt/mcp_services/*.py" -type f | xargs python -m mypy --strict 2>&1 | head -20

# Discover the project's linting configuration and verify rules detect stdlib logging imports in the MCP services layer
grep -r "import logging" "$(find . -path '*/tlt/mcp_services/*.py' -type f)" || echo "No stdlib logging imports found (pass)"

# Verify all service modules use os.getenv for configuration
grep -r "os\.getenv" "$(find . -path '*/tlt/mcp_services/*.py' -type f)" | wc -l
```

**Accept when:**
- All MCP service modules import and use the standardized library stack (os, typing, datetime, uuid, json, loguru) without importing stdlib logging
- Static type checker reports no type errors in service module function signatures
- Linting passes with no violations for stdlib logging usage in service layer
- Configuration retrieval uses `os.getenv` with appropriate default values throughout service modules
- Timestamps use `datetime.now(timezone.utc)` for timezone-aware UTC handling
- Unique identifiers are generated using UUID v4

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules marked MUST are mandatory and must be verified before accepting code changes to MCP service modules. CI pipeline automation MUST enforce these rules.
</enforcement>