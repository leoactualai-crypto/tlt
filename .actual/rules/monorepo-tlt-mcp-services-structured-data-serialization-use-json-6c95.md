# Adopt Standardized Core Library Stack for Python MCP Services: Structured Data Serialization Use Json Module

These rules are ALWAYS ACTIVE for all Python service modules in the MCP services layer (`monorepo/tlt/mcp_services/*`) that manage state, interact with external systems, or require observability.

### Rules

- **R-STDLIB-001** SHOULD: Structured data serialization SHOULD use the `json` module for consistency with external API contracts.
- **R-STDLIB-002** MUST: When creating new MCP service modules, import the standardized library stack at the top of the file following Python import conventions: stdlib imports first (`os`, `typing`, `datetime`, `uuid`, `json`), then third-party imports (`loguru`).
- **R-STDLIB-003** MUST: Configure loguru logger at service initialization to establish consistent log formatting, levels, and output destinations across all service modules.
- **R-STDLIB-004** SHOULD: Use `typing.Optional`, `typing.List`, `typing.Dict` for nullable and collection types in function signatures to maximize type checker effectiveness.
- **R-STDLIB-005** MUST: Do not import Python standard library `logging` module in MCP service layer; use `loguru` for all structured logging.

### Verify

```bash
# Discover the project's dependency manifest and verify loguru is declared as a dependency for the MCP services package
grep -r "loguru" "$(find . -name 'requirements*.txt' -o -name 'pyproject.toml' -o -name 'setup.py' | head -1)"

# Discover the project's static analysis configuration and run the type checker against service modules to verify type annotation coverage
find . -path "*/mcp_services/*.py" -type f | xargs python -m mypy --strict 2>&1 | grep -E "(error|success)"

# Discover the project's linting configuration and verify rules detect stdlib logging imports in the MCP services layer
find . -path "*/mcp_services/*.py" -type f -exec grep -l "^import logging\|^from logging" {} \; | wc -l
```

**Accept when:**
- All MCP service modules import and use the standardized library stack (`os`, `typing`, `datetime`, `uuid`, `json`, `loguru`) without importing stdlib `logging`
- Static type checker reports no type errors in service module function signatures
- Linting passes with no violations for stdlib logging usage in service layer
- Loguru is declared as a dependency in the project's dependency manifest
- Service module templates pre-populate standard library imports in the correct order

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and CI pipeline enforcement.
</enforcement>