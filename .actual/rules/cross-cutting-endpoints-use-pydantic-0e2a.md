# Enforce Input Validation and HTTP Exception Handling in Discord Integration Endpoints: Endpoints Use Pydantic

These rules are ALWAYS ACTIVE for all HTTP endpoint handlers in the Discord adapter that access bot state or make Discord API calls.

### Rules

- **R-DISCORD-001** MUST: Endpoints MUST use Pydantic BaseModel subclasses for request body validation with explicit type annotations for message_id (int), emoji (str), user_id (int), and action (str) fields.
- **R-DISCORD-002** MUST: All HTTP endpoint handlers MUST validate message_id existence in bot.active_events before proceeding with Discord API calls.
- **R-DISCORD-003** MUST: Endpoints MUST raise HTTPException with status_code=404 for missing resources (events, guild, channel, message, user) and status_code=500 for system exceptions.
- **R-DISCORD-004** MUST: Nested state structures MUST be defensively initialized before mutation operations (e.g., if 'reactions' not in event: event['reactions'] = {}).
- **R-DISCORD-005** SHOULD: Validation failures triggered by user input (user not found, event not found) SHOULD be logged at info level; system exceptions SHOULD be logged at error level.

### Verify

```bash
# Verify Pydantic BaseModel definitions exist
grep -r 'class.*BaseModel' monorepo/tlt/adapters/discord_adapter/ | grep -v '__pycache__'

# Verify HTTPException usage in endpoints
grep -r 'raise HTTPException' monorepo/tlt/adapters/discord_adapter/rsvp.py | wc -l

# Verify bot.active_events validation checks
grep -r 'if.*not in bot.active_events' monorepo/tlt/adapters/discord_adapter/rsvp.py
```

**Accept when:**
- All HTTP endpoint handlers validate message_id existence in bot.active_events before Discord API calls
- Request bodies use Pydantic BaseModel validation and handlers raise HTTPException for missing resources (404) and exceptions (500)
- Nested state structures are defensively initialized before mutation operations
- Integration tests cover validation failure paths (missing events, missing Discord resources, malformed requests)

<enforcement>
Clause Code MUST NOT skip or defer verification. Pull requests adding HTTP endpoints without input validation or exception handling are blocked in code review. Integration test failures for validation paths block merge to main branch.
</enforcement>