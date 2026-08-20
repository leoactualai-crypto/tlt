# Adopt Standardized Core Library Stack for Python MCP Services: Service Modules Use Loguru Logger Instance

These rules are ALWAYS ACTIVE for all Python service modules in the MCP services layer (`monorepo/tlt/mcp_services/*`) that manage state, interact with external systems, or require observability.

### Rules

- **R-LOGURU-001** MUST: Service modules MUST use loguru's logger instance for all logging operations rather than Python's standard library logging module.
- **R-LOGURU-002** MUST: Service modules MUST import the standardized library stack at the top of the file following Python import conventions: stdlib imports first (os, typing, datetime, uuid, json), then third-party imports (loguru).
- **R-LOGURU-003** MUST: Service modules MUST configure loguru logger at service initialization to establish consistent log formatting, levels, and output destinations.
- **R-LOGURU-004** SHOULD: Service modules SHOULD use typing.Optional, typing.List, typing.Dict for nullable and collection types in function signatures to maximize type checker effectiveness.
- **R-LOGURU-005** SHOULD: Service modules SHOULD use timezone-aware UTC timestamps (datetime.now(timezone.utc)) and UUID v4 for identifiers to establish consistent data handling patterns.
- **R-LOGURU-006** SHOULD: Service modules SHOULD use os.getenv for environment-based configuration to align with 12-factor app methodology.

### Verify

```bash
# Discover the project's dependency manifest and verify loguru is declared as a dependency for the MCP services package
grep -r "loguru" "$(find . -name 'pyproject.toml' -o -name 'requirements*.txt' -o -name 'poetry.lock' -o -name 'Pipfile.lock' | head -1)"

# Discover the project's static analysis configuration and run the type checker against service modules
find . -path '*/tlt/mcp_services/*.py' -type f | xargs python -m mypy --strict 2>&1 | head -20

# Discover the project's linting configuration and verify rules detect stdlib logging imports in the MCP services layer
grep -r "import logging" monorepo/tlt/mcp_services/ 2>/dev/null || echo "No stdlib logging imports found (pass)"
```

**Accept when:**
- All MCP service modules import and use the standardized library stack (os, typing, datetime, uuid, json, loguru) without importing stdlib logging
- Static type checker reports no type errors in service module function signatures
- Linting passes with no violations for stdlib logging usage in service layer
- loguru is declared as a dependency in the project's dependency manifest

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code in scope. Violations block merge and trigger refactoring tickets.
</enforcement>