# Adopt Async Real-Time Messaging for Discord Integration Boundaries: Adapter Implementations Use

These rules are ALWAYS ACTIVE for all Discord adapter implementations and real-time messaging boundaries within the TLT service integration layer, including all files in `monorepo/tlt/adapters/discord_adapter/`, FastAPI router endpoints that trigger Discord messaging operations, bot lifecycle hooks, reminder scheduling workflows, and state synchronization tasks between Discord and TLT service backend.

### Rules

- **R-DISCORD-ASYNC-001** SHOULD: Adapter implementations SHOULD use structured logging with module-level loggers (`logging.getLogger(__name__)`) to trace async operations across integration boundaries.

### Verify

```bash
# Verify async messaging operations use await keyword
grep -r 'await.*\.send(' monorepo/tlt/adapters/discord_adapter/ | wc -l

# Verify async function definitions for key operations
grep -r 'async def' monorepo/tlt/adapters/discord_adapter/ | grep -E '(create_reminder|schedule_reminder|setup_hook|on_ready)' | wc -l

# Verify async HTTP client usage
grep -r 'AsyncClient\|ClientSession' monorepo/tlt/adapters/discord_adapter/ | wc -l

# Verify Pydantic BaseModel schemas at boundaries
grep -r 'class.*BaseModel' monorepo/tlt/adapters/discord_adapter/ | grep -E '(ReminderCreate|ReminderResponse)' | wc -l
```

**Accept when:**
- All Discord messaging operations (channel.send, thread.send, user.send) are prefixed with 'await' keyword in adapter modules
- All public API contracts in Discord adapter modules use 'async def' function signatures
- External HTTP client usage shows AsyncClient or ClientSession patterns rather than synchronous requests
- Pydantic BaseModel schemas are defined for data structures crossing integration boundaries (ReminderCreate, ReminderResponse)

<enforcement>
Claude Code MUST NOT skip or defer verification. Automated code review checks using pylint-asyncio MUST detect blocking operations in async functions. CI pipeline MUST fail if verification commands do not meet acceptance criteria thresholds. Code review MUST block merge if blocking operations are detected in async code paths without justification. Runtime monitoring of event loop lag metrics MUST be implemented to detect blocking operations in production.
</enforcement>