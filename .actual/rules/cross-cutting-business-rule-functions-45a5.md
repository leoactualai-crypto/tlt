# Adopt Async Function Definitions for Business Rule Orchestration in FastAPI Service Endpoints: Business Rule Functions

These rules are ALWAYS ACTIVE for all FastAPI service endpoints that orchestrate business rules, handle external I/O operations, or coordinate Discord bot interactions within the TLT monorepo.

### Rules

- **R-ASYNC-001** MAY: Business rule functions MAY coordinate multiple async operations concurrently using asyncio.gather or similar patterns when operations are independent.

### Verify

```bash
# Verify async function definitions in business rule handlers
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

<enforcement>
Claude Code MUST NOT skip or defer verification. All async function definitions in business rule handlers MUST be verified against the rules and acceptance criteria before approval.
</enforcement>