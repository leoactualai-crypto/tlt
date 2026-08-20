# Adopt Standardized Core Library Stack for Python MCP Services: Timestamp Generation Use Datetime Timezone Aware

These rules are ALWAYS ACTIVE for all Python service modules in the MCP services layer (monorepo/tlt/mcp_services/*) that manage state, interact with external systems, or require observability.

### Rules

- **R-CORE-001** MUST: Timestamp generation MUST use datetime with timezone-aware UTC timestamps via `datetime.now(timezone.utc)`.
- **R-CORE-002** MUST: All MCP service modules MUST import the standardized library stack in this order: stdlib imports first (os, typing, datetime, uuid, json), then third-party imports (loguru).
- **R-CORE-003** MUST: Service modules MUST NOT import Python standard library logging module; use loguru for all structured logging.
- **R-CORE-004** SHOULD: Function signatures SHOULD use typing.Optional, typing.List, typing.Dict for nullable and collection types to maximize type checker effectiveness.
- **R-CORE-005** SHOULD: Service initialization SHOULD configure loguru logger to establish consistent log formatting, levels, and output destinations.

### Verify

```bash
# Discover the project's dependency manifest and verify loguru is declared as a dependency for the MCP services package
grep -r "loguru" pyproject.toml requirements.txt setup.py 2>/dev/null || echo "Dependency manifest check required"

# Discover the project's static analysis configuration and run the type checker against service modules
python -m mypy monorepo/tlt/mcp_services/ --strict 2>/dev/null || echo "Type checker verification required"

# Discover the project's linting configuration and verify rules detect stdlib logging imports in the MCP services layer
grep -r "import logging" monorepo/tlt/mcp_services/ && echo "FAIL: stdlib logging detected" || echo "PASS: no stdlib logging imports"

# Verify timezone-aware UTC timestamp usage
grep -r "datetime.now(timezone.utc)" monorepo/tlt/mcp_services/ || echo "Timestamp verification required"
```

**Accept when:**
- All MCP service modules import and use the standardized library stack (os, typing, datetime, uuid, json, loguru) without importing stdlib logging
- Static type checker reports no type errors in service module function signatures
- Linting passes with no violations for stdlib logging usage in service layer
- Timestamp generation consistently uses `datetime.now(timezone.utc)` across all service modules
- Service module templates pre-populate standard library imports in correct order

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for MCP service layer code. Type checking and linting verification MUST pass before code review approval.
</enforcement>