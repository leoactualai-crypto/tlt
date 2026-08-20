# Adopt Standardized Core Library Stack for Python MCP Services: Service Module Functions Include Type Annotations

These rules are ALWAYS ACTIVE for all Python service modules in the MCP services layer (`monorepo/tlt/mcp_services/*`) that manage state, interact with external systems, or require observability.

### Rules

- **R-STDLIB-001** SHOULD: Service module functions SHOULD include type annotations using the `typing` module for parameters and return values.
- **R-STDLIB-002** MUST: Service modules MUST import the standardized library stack in this order: stdlib imports first (`os`, `typing`, `datetime`, `uuid`, `json`), then third-party imports (`loguru`).
- **R-STDLIB-003** MUST: Service modules MUST NOT import Python's standard library `logging` module; use `loguru` for all structured logging.
- **R-STDLIB-004** SHOULD: Service modules SHOULD use `typing.Optional`, `typing.List`, `typing.Dict` for nullable and collection types in function signatures to maximize type checker effectiveness.
- **R-STDLIB-005** SHOULD: Service modules SHOULD use `datetime.now(timezone.utc)` for timezone-aware UTC timestamps.
- **R-STDLIB-006** SHOULD: Service modules SHOULD use `uuid.uuid4()` for unique identifier generation.
- **R-STDLIB-007** SHOULD: Service modules SHOULD use `os.getenv()` for environment-based configuration following 12-factor app principles.

### Verify

```bash
# Discover the project's dependency manifest and verify loguru is declared as a dependency
grep -r "loguru" pyproject.toml requirements.txt setup.py 2>/dev/null || echo "Dependency manifest check required"

# Discover the project's static analysis configuration and run type checker
if [ -f "pyproject.toml" ] || [ -f "setup.cfg" ] || [ -f "mypy.ini" ]; then
  echo "Type checker configuration found"
fi

# Verify no stdlib logging imports in MCP service modules
grep -r "import logging" monorepo/tlt/mcp_services/ 2>/dev/null && echo "FAIL: stdlib logging found" || echo "PASS: No stdlib logging imports"

# Verify type annotations are present in service module functions
grep -r "def .*->" monorepo/tlt/mcp_services/*.py 2>/dev/null | head -5 || echo "Type annotation check required"

# Verify loguru imports are present
grep -r "from loguru import" monorepo/tlt/mcp_services/ 2>/dev/null | head -5 || echo "Loguru import check required"
```

**Accept when:**
- All MCP service modules import and use the standardized library stack (`os`, `typing`, `datetime`, `uuid`, `json`, `loguru`) without importing stdlib `logging`
- Static type checker reports no type errors in service module function signatures
- Linting passes with no violations for stdlib logging usage in service layer
- Service module functions include type annotations for parameters and return values
- Import order follows Python conventions: stdlib first, then third-party

<enforcement>
Claude Code MUST NOT skip or defer verification. Type checking and linting must pass in CI before code is merged. Violations trigger build failure and code review rejection.
</enforcement>