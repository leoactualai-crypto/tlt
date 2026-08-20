# Adopt Standardized Core Library Stack for Python MCP Services: Mcp Service Modules Use Standardized Core

These rules are ALWAYS ACTIVE for all Python service modules in the MCP services layer (`monorepo/tlt/mcp_services/*`) that manage state, interact with external systems, or require observability.

### Rules

- **R-MCP-CORE-001** MUST: MCP service modules MUST import and use the standardized core library stack: `os` for environment access, `typing` for type annotations, `datetime` for temporal operations, `uuid` for unique identifier generation, `json` for data serialization, and `loguru` for structured logging.
- **R-MCP-CORE-002** MUST: Service modules MUST NOT import Python's standard library `logging` module; use `loguru` exclusively for structured logging in the MCP services layer.
- **R-MCP-CORE-003** MUST: When creating new MCP service modules, import the standardized library stack at the top of the file following Python import conventions: stdlib imports first (`os`, `typing`, `datetime`, `uuid`, `json`), then third-party imports (`loguru`).
- **R-MCP-CORE-004** MUST: Configure `loguru` logger at service initialization to establish consistent log formatting, levels, and output destinations across all service modules.
- **R-MCP-CORE-005** SHOULD: Use `typing.Optional`, `typing.List`, `typing.Dict` for nullable and collection types in function signatures to maximize type checker effectiveness.
- **R-MCP-CORE-006** SHOULD: Use timezone-aware UTC timestamps via `datetime.now(timezone.utc)` and UUID v4 for identifiers to establish consistent data handling patterns across distributed service components.
- **R-MCP-CORE-007** SHOULD: Use `os.getenv()` for environment-based configuration to align with 12-factor app methodology and enable environment-specific configuration without code changes.

### Verify

```bash
# 1. Discover the project's dependency manifest and verify loguru is declared as a dependency for the MCP services package
grep -r "loguru" "$(find . -name 'pyproject.toml' -o -name 'requirements*.txt' -o -name 'setup.py' | head -1 | xargs dirname)"

# 2. Discover the project's static analysis configuration and run the type checker against service modules
find . -path "*/tlt/mcp_services/*.py" -type f | head -5

# 3. Discover the project's linting configuration and verify rules detect stdlib logging imports in the MCP services layer
grep -r "import logging" "./tlt/mcp_services/" || echo "No stdlib logging imports found (pass)"
```

**Accept when:**
- All MCP service modules in `monorepo/tlt/mcp_services/*` import and use the standardized library stack (`os`, `typing`, `datetime`, `uuid`, `json`, `loguru`) without importing stdlib `logging`
- Static type checker reports no type errors in service module function signatures
- Linting passes with no violations for stdlib `logging` usage in the MCP services layer
- `loguru` is declared as a dependency in the project's dependency manifest
- Service module templates pre-populate standard library imports in the correct order

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for MCP service modules. CI build MUST fail if linting detects stdlib logging imports in the service layer or if type checker reports errors. Code review MUST block merge if non-standard libraries are introduced without architectural justification.
</enforcement>