# Enforce Input Validation and HTTP Exception Handling in Discord Integration Endpoints: Handlers Wrap Discord

These rules are ALWAYS ACTIVE for all HTTP endpoint handlers in the Discord adapter that access bot state or invoke Discord API operations.

### Rules

- **R-DISCORD-001** MUST: Handlers MUST wrap Discord API operations in try-except blocks and raise HTTPException with status_code=500 and error details on exception.
- **R-DISCORD-002** MUST: All HTTP endpoint handlers MUST validate message_id existence in bot.active_events before proceeding with Discord API calls.
- **R-DISCORD-003** MUST: Request bodies MUST use Pydantic BaseModel validation and handlers MUST raise HTTPException with status_code=404 for missing resources.
- **R-DISCORD-004** MUST: Nested state structures MUST be defensively initialized before mutation operations (e.g., if 'reactions' not in event: event['reactions'] = {}).
- **R-DISCORD-005** SHOULD: Validation failures triggered by user input SHOULD be logged at info level; system exceptions SHOULD be logged at error level.

### Verify

```bash
# Verify Pydantic BaseModel definitions exist for request validation
grep -r 'class.*BaseModel' monorepo/tlt/adapters/discord_adapter/ | grep -v '__pycache__'

# Verify HTTPException usage in handlers
grep -r 'raise HTTPException' monorepo/tlt/adapters/discord_adapter/rsvp.py | wc -l

# Verify bot.active_events validation pattern
grep -r 'if.*not in bot.active_events' monorepo/tlt/adapters/discord_adapter/rsvp.py
```

**Accept when:**
- All HTTP endpoint handlers validate message_id existence in bot.active_events before Discord API calls
- Request bodies use Pydantic BaseModel validation and handlers raise HTTPException for missing resources (404) and exceptions (500)
- Nested state structures are defensively initialized before mutation operations
- Try-except blocks wrap all Discord API operations with appropriate HTTPException responses

<enforcement>
Clause Code MUST NOT skip or defer verification of these rules. All new HTTP endpoints accessing bot state or Discord API MUST satisfy R-DISCORD-001 through R-DISCORD-005 before merge.
</enforcement>