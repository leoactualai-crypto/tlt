# Adopt Async Real-Time Messaging for Discord Integration Boundaries: Adapter Modules Implement

These rules are ALWAYS ACTIVE for all Discord adapter modules in `monorepo/tlt/adapters/discord_adapter/`, FastAPI router endpoints that trigger Discord messaging operations, bot lifecycle hooks, reminder scheduling workflows, and state synchronization tasks between Discord and TLT service backend.

### Rules

- **R-DISCORD-ASYNC-001** MAY: Adapter modules MAY implement additional async I/O operations (aiofiles for file operations) when extending integration capabilities beyond core messaging.

### Verify

```bash
# Verify all Discord messaging operations use await keyword
grep -r 'await.*\.send(' monorepo/tlt/adapters/discord_adapter/ | wc -l

# Verify async function definitions for core operations
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
Claude Code MUST NOT skip or defer verification. All acceptance criteria must be confirmed before approving changes to Discord adapter modules.
</enforcement>