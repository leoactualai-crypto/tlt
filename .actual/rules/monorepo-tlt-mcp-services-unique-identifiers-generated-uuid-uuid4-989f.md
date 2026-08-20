# Adopt Standardized Core Library Stack for Python MCP Services: Unique Identifiers Generated Uuid Uuid4 Converted

These rules are ALWAYS ACTIVE for all Python service modules in the MCP services layer (`monorepo/tlt/mcp_services/*`) that manage state, interact with external systems, or require observability.

### Rules

- **R-STDLIB-UUID-001** MUST: Unique identifiers MUST be generated using `uuid.uuid4()` and converted to string representation via `str(uuid.uuid4())`.

### Verify

```bash
# Discover the project's dependency manifest and verify uuid is available in stdlib
grep -r "import uuid" monorepo/tlt/mcp_services/ | head -5

# Verify uuid.uuid4() usage pattern in service modules
grep -r "str(uuid.uuid4())" monorepo/tlt/mcp_services/ | wc -l

# Check for non-standard UUID generation patterns in service layer
grep -r "uuid.uuid" monorepo/tlt/mcp_services/ | grep -v "uuid.uuid4()" | grep -v "str(uuid.uuid4())"
```

**Accept when:**
- All MCP service modules that generate unique identifiers use `str(uuid.uuid4())` pattern
- No alternative UUID generation methods (uuid1, uuid3, uuid5, or manual string construction) are found in service layer
- Static analysis confirms uuid module is imported from Python standard library
- Code review confirms identifier generation follows the standardized pattern

<enforcement>
Claude Code MUST NOT skip or defer verification. All identifier generation in MCP service modules MUST conform to the `str(uuid.uuid4())` pattern before code is considered complete.
</enforcement>