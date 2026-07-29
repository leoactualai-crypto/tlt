# Adopt Async Function Definitions for Business Rule Orchestration in FastAPI Service Endpoints: Business Rule Orchestration

These rules are ALWAYS ACTIVE for all FastAPI service endpoints that orchestrate business rules, handle external I/O operations, or coordinate Discord bot interactions within the TLT monorepo.

### Rules

- **R-ASYNC-001** SHOULD: Business rule orchestration functions SHOULD be named with verb-noun patterns (create_reminder, handle_reaction, process_photo) to indicate their action-oriented nature.
- **R-ASYNC-002** MUST: All FastAPI router endpoint handlers in discord_adapter modules (reminder.py, experience_manager.py, rsvp.py) and MCP service business rule processors MUST be defined with async def syntax.
- **R-ASYNC-003** MUST: All Discord API calls (channel.fetch_message, thread.send, discord.utils.get) MUST use await and be coordinated within async function contexts.
- **R-ASYNC-004** MUST: Pydantic BaseModel validation MUST be applied before async business rule execution in all endpoints.
- **R-ASYNC-005** MUST: No synchronous blocking I/O operations (requests.get without timeout, blocking file I/O) are permitted in async function bodies without explicit exception approval.
- **R-ASYNC-006** SHOULD: HTTP requests within async functions SHOULD use aiohttp or equivalent async HTTP client instead of synchronous requests library.
- **R-ASYNC-007** SHOULD: File I/O operations within async functions SHOULD use aiofiles or equivalent async file operations instead of synchronous file I/O.

### Verify

```bash
# Verify async function definitions for business rule orchestration
grep -r 'async def' monorepo/tlt/adapters/discord_adapter/ monorepo/tlt/services/tlt_service/ monorepo/tlt/mcp_services/ | grep -E '(create_|handle_|process_|get_|delete_)' | wc -l

# Verify all router handlers are async
grep -r '@router\.(post|get|delete|put)' monorepo/tlt/adapters/discord_adapter/ monorepo/tlt/services/tlt_service/ | grep -v 'async def' && echo 'Found synchronous router handlers' || echo 'All router handlers are async'

# Verify no blocking requests without timeout in async contexts
grep -r 'requests\.get\|requests\.post' monorepo/tlt/adapters/discord_adapter/ monorepo/tlt/mcp_services/ | grep -v 'timeout=' && echo 'Found blocking requests without timeout' || echo 'All requests have timeout or use async client'
```

**Accept when:**
- All FastAPI router endpoint handlers in discord_adapter and service modules are defined with async def syntax
- No synchronous blocking I/O operations (requests.get without timeout, blocking file I/O) are present in async function bodies
- All Discord API calls use await and are coordinated within async function contexts
- Pydantic BaseModel validation is applied before async business rule execution in all endpoints
- Business rule orchestration functions follow verb-noun naming patterns

<enforcement>
Claude Code MUST NOT skip or defer verification. CI pipeline linting with ruff or pylint MUST be configured to detect blocking calls in async functions. Code review MUST verify async/await patterns in business rule handlers. Pre-commit hooks MUST check for synchronous router handlers in FastAPI service modules. Violations result in CI build failure and code review blocking.
</enforcement>