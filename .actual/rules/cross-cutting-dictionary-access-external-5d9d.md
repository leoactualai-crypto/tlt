# Adopt .get() Dictionary Access Pattern for Integration Testing and External API Response Handling: Dictionary Access External

These rules are ALWAYS ACTIVE for all internal API implementations, integration test suites, and external client boundary code within the monorepo, including MCP service tools, Discord adapter command handlers, external client implementations, integration test code, photo processor and workflow state handling, and agent state management.

### Rules

- **R-DICT-001** MUST: All dictionary access for external API responses, integration test assertions, and inter-service communication MUST use the .get() method rather than direct key access (dict['key']).

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
- Direct dictionary key access count in service and adapter code is less than 5% of total dictionary accesses (excluding Pydantic model access)
- .get() method usage count for response/result/event/state dictionaries exceeds 95% of accesses in integration boundary code
- Integration test suite runs without KeyError exceptions when external services return partial or error responses

<enforcement>
Clause Code MUST NOT skip or defer verification. Violations detected by CI pipeline grep patterns or integration test KeyError exceptions MUST be remediated before merge. Exceptions require explicit code comments documenting why direct key access is safe (Pydantic model field or validated guard clause) and code reviewer approval.
</enforcement>