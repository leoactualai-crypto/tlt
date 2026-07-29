# Adopt Async Function Definitions for Business Rule Orchestration in FastAPI Service Endpoints: Business Rule Handlers

These rules are ALWAYS ACTIVE for all FastAPI service endpoints that orchestrate business rules, handle external I/O operations, or coordinate Discord bot interactions within the TLT monorepo, including discord_adapter modules (reminder.py, experience_manager.py, rsvp.py), MCP service business rule processors (photo_processor.py), and service monitoring endpoints (monitor.py).

### Rules

- **R-ASYNC-001** MUST NOT: Business rule handlers MUST NOT use blocking I/O operations (synchronous requests.get without timeout, blocking file I/O) within async function bodies.
- **R-ASYNC-002** MUST: All FastAPI router endpoint handlers in discord_adapter and service modules MUST be defined with async def syntax.
- **R-ASYNC-003** MUST: All Discord API calls (channel.fetch_message, thread.send, discord.utils.get) MUST use await and be coordinated within async function contexts.
- **R-ASYNC-004** MUST: Pydantic BaseModel validation MUST be applied before async business rule execution in all endpoints.
- **R-ASYNC-005** SHOULD: Use aiohttp for HTTP requests and aiofiles for file I/O instead of synchronous alternatives in async functions.
- **R-ASYNC-006** SHOULD: Implement structured logging with logger.info, logger.error at key async execution points for observability.

### Verify

```bash
# Verify async function definitions for business rule handlers
grep -r 'async def' monorepo/tlt/adapters/discord_adapter/ monorepo/tlt/services/tlt_service/ monorepo/tlt/mcp_services/ | grep -E '(create_|handle_|process_|get_|delete_)' | wc -l

# Verify all router handlers are async
grep -r '@router\.(post|get|delete|put)' monorepo/tlt/adapters/discord_adapter/ monorepo/tlt/services/tlt_service/ | grep -v 'async def' && echo 'Found synchronous router handlers' || echo 'All router handlers are async'

# Verify no blocking requests without timeout
grep -r 'requests\.get\|requests\.post' monorepo/tlt/adapters/discord_adapter/ monorepo/tlt/mcp_services/ | grep -v 'timeout=' && echo 'Found blocking requests without timeout' || echo 'All requests have timeout or use async client'
```

**Accept when:**
- All FastAPI router endpoint handlers in discord_adapter and service modules are defined with async def syntax
- No synchronous blocking I/O operations (requests.get without timeout, blocking file I/O) are present in async function bodies
- All Discord API calls use await and are coordinated within async function contexts
- Pydantic BaseModel validation is applied before async business rule execution in all endpoints
- HTTP requests use aiohttp or include timeout parameters
- Structured logging is present at key async execution points

<enforcement>
Claude Code MUST NOT skip or defer verification of these rules. All async function definitions in business rule handlers MUST be reviewed for blocking I/O operations, and all FastAPI router handlers MUST be confirmed as async. Violations MUST be flagged during code review and CI pipeline checks MUST fail if synchronous router handlers or blocking I/O in async functions are detected.
</enforcement>