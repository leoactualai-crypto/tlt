# Adopt Async Function Definitions for Business Rule Orchestration in FastAPI Service Endpoints: Pydantic Basemodel Validation

These rules are ALWAYS ACTIVE for all FastAPI service endpoints that orchestrate business rules, handle external I/O operations, or coordinate Discord bot interactions within the TLT monorepo.

### Rules

- **R-ASYNC-001** MUST: Pydantic BaseModel validation classes used for request/response contracts MUST be defined synchronously and applied before async business rule execution.
- **R-ASYNC-002** MUST: All FastAPI router endpoint handlers in discord_adapter modules (reminder.py, experience_manager.py, rsvp.py) and MCP service business rule processors (photo_processor.py) MUST be defined with async def syntax.
- **R-ASYNC-003** MUST: All Discord API calls (channel.fetch_message, thread.send, discord.utils.get) within async functions MUST use await and be coordinated within async function contexts.
- **R-ASYNC-004** MUST: No synchronous blocking I/O operations (requests.get without timeout, blocking file I/O) are permitted in async function bodies without exception approval.
- **R-ASYNC-005** SHOULD: Use aiohttp for HTTP requests and aiofiles for file I/O within async functions to maintain non-blocking execution.
- **R-ASYNC-006** SHOULD: Implement structured logging with logger.info, logger.error at key async execution points for observability.
- **R-ASYNC-007** SHOULD: Wrap all async endpoint handlers with try-except blocks and use FastAPI exception handlers to prevent event loop crashes.

### Verify

```bash
# Count async business rule functions
grep -r 'async def' monorepo/tlt/adapters/discord_adapter/ monorepo/tlt/services/tlt_service/ monorepo/tlt/mcp_services/ | grep -E '(create_|handle_|process_|get_|delete_)' | wc -l

# Verify all router handlers are async
grep -r '@router\.(post|get|delete|put)' monorepo/tlt/adapters/discord_adapter/ monorepo/tlt/services/tlt_service/ | grep -v 'async def' && echo 'Found synchronous router handlers' || echo 'All router handlers are async'

# Check for blocking requests without timeout
grep -r 'requests\.get\|requests\.post' monorepo/tlt/adapters/discord_adapter/ monorepo/tlt/mcp_services/ | grep -v 'timeout=' && echo 'Found blocking requests without timeout' || echo 'All requests have timeout or use async client'
```

**Accept when:**
- All FastAPI router endpoint handlers in discord_adapter and service modules are defined with async def syntax
- No synchronous blocking I/O operations (requests.get without timeout, blocking file I/O) are present in async function bodies
- All Discord API calls use await and are coordinated within async function contexts
- Pydantic BaseModel validation is applied before async business rule execution in all endpoints
- Exceptions (EXC-001, EXC-002) are documented in function docstrings with rationale and mitigation strategy

<enforcement>
Claude Code MUST NOT skip or defer verification. CI pipeline linting with ruff or pylint MUST be configured to detect blocking calls in async functions. Code review MUST block merge if blocking I/O operations are found in async function bodies without justification. Runtime monitoring MUST alert if event loop blocking is detected (execution time > 100ms for single async operation).
</enforcement>