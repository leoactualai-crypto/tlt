# Adopt .get() Dictionary Access Pattern for Integration Testing and External API Response Handling: When Extracting Nested

These rules are ALWAYS ACTIVE for all MCP service tools, Discord adapter command handlers, external client implementations, integration test code, photo processor and workflow state handling, and agent state management code that accesses dictionary responses from internal APIs and external service boundaries.

### Rules

- **R-DICT-001** SHOULD: When extracting nested dictionary values, code SHOULD chain .get() calls (e.g., `state.get('lifecycles', {}).get(task_id)`) to prevent AttributeError on None.
- **R-DICT-002** MUST: For external API responses (httpx, requests), always use .get() with defaults that enable error path execution: `result.get('success', False)`, `result.get('error', 'Unknown error')`.
- **R-DICT-003** SHOULD: In integration tests, use .get() with explicit defaults to document expected response structure: `assert response.get('status') == 'success'`.
- **R-DICT-004** SHOULD: When logging response data, use .get() to prevent log formatting failures: `logger.info(f"Guild: {result.get('guild_id', 'unknown')}")`.
- **R-DICT-005** SHOULD: For CloudEvent data extraction, use .get() since event data structure varies by event type: `event.get('guild_id')`, `event.get('message_id')`.
- **R-DICT-006** MAY: Direct key access is permitted when the dictionary is a Pydantic model's .dict() or .model_dump() output and the field is required (not Optional).
- **R-DICT-007** MAY: Direct key access is permitted for dictionary keys that are explicitly validated with `if key in dict` guard immediately before access.

### Verify

```bash
# Count direct dictionary key access patterns (excluding .get() usage)
grep -r "\['[a-zA-Z_]*'\]" monorepo/tlt/mcp_services --include='*.py' | grep -v test | grep -v '.get(' | wc -l

# Count .get() method usage for response/result/event/state dictionaries
grep -r "\.get(" monorepo/tlt/mcp_services monorepo/tlt/adapters/discord_adapter --include='*.py' | grep -E "(response|result|event|data|state)\.get\(" | wc -l

# Verify integration test suite runs without KeyError exceptions
python -m pytest monorepo/tlt/tests/integration -v --tb=short 2>&1 | grep -i keyerror | wc -l
```

**Accept when:**
- Direct dictionary key access count in service and adapter code is less than 5% of total dictionary accesses (excluding Pydantic model access).
- .get() method usage count for response/result/event/state dictionaries exceeds 95% of accesses in integration boundary code.
- Integration test suite runs without KeyError exceptions when external services return partial or error responses.

<enforcement>
Clause Code MUST NOT skip or defer verification. CI pipeline MUST fail if grep detects direct dictionary key access patterns in new code touching external API boundaries. Code review MUST block merge if direct key access is used without explicit guard clause or documented exception. Production KeyError exceptions MUST trigger incident review to add .get() usage and integration test coverage.
</enforcement>