# Enforce Input Validation and HTTP Exception Handling in Discord Integration Endpoints: Handlers Raise Httpexception

These rules are ALWAYS ACTIVE for all HTTP endpoint handlers in the Discord adapter that access bot state or Discord API resources.

### Rules

- **R-DISCORD-001** MUST: Handlers MUST raise HTTPException with status_code=404 when referenced resources (event, guild, channel, message, user) are not found
- **R-DISCORD-002** MUST: All HTTP endpoint handlers MUST validate message_id existence in bot.active_events before making Discord API calls
- **R-DISCORD-003** MUST: Request bodies MUST use Pydantic BaseModel validation for type safety and automatic request parsing
- **R-DISCORD-004** MUST: Handlers MUST raise HTTPException with status_code=500 for unhandled exceptions during Discord API operations
- **R-DISCORD-005** MUST: Nested state structures MUST be defensively initialized before mutation operations (e.g., if 'reactions' not in event: event['reactions'] = {})
- **R-DISCORD-006** SHOULD: Validation failures for user-triggered conditions SHOULD be logged at info level; system exceptions SHOULD be logged at error level
- **R-DISCORD-007** SHOULD: Common validation patterns SHOULD be extracted into shared helper functions or FastAPI dependencies to reduce duplication

### Verify

```bash
# Verify Pydantic BaseModel definitions exist
grep -r 'class.*BaseModel' monorepo/tlt/adapters/discord_adapter/ | grep -v '__pycache__'

# Count HTTPException raises in rsvp.py
grep -r 'raise HTTPException' monorepo/tlt/adapters/discord_adapter/rsvp.py | wc -l

# Verify bot.active_events validation pattern
grep -r 'if.*not in bot.active_events' monorepo/tlt/adapters/discord_adapter/rsvp.py
```

**Accept when:**
- All HTTP endpoint handlers validate message_id existence in bot.active_events before Discord API calls
- Request bodies use Pydantic BaseModel validation and handlers raise HTTPException for missing resources (404) and exceptions (500)
- Nested state structures are defensively initialized before mutation operations
- Validation failures are logged at appropriate levels (info for user-triggered, error for system exceptions)

<enforcement>
Clause Code MUST NOT skip or defer verification. All new HTTP endpoints accessing bot state or Discord API MUST pass these rules before merge. Integration test failures for validation paths block merge to main branch. Production incidents caused by unhandled exceptions trigger post-mortem review and enforcement.
</enforcement>