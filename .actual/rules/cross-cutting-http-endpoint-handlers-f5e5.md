# Enforce Input Validation and HTTP Exception Handling in Discord Integration Endpoints: Http Endpoint Handlers

These rules are ALWAYS ACTIVE for all HTTP endpoint handlers in the Discord adapter that accept message_id parameters, coordinate between FastAPI router handlers and Discord bot client operations, or mutate bot state structures.

### Rules

- **R-DISCORD-HTTP-001** MUST: All HTTP endpoint handlers accepting message_id parameters MUST validate the message_id exists in bot.active_events before performing Discord API operations.
- **R-DISCORD-HTTP-002** MUST: All HTTP endpoint handlers accepting external requests MUST use Pydantic BaseModel for request body validation and raise HTTPException with appropriate status codes (404 for not found, 500 for server errors).
- **R-DISCORD-HTTP-003** MUST: All state mutation operations on nested dictionaries (event['reactions'], event['reactions'][emoji]) MUST defensively initialize parent structures before access to prevent KeyError exceptions.
- **R-DISCORD-HTTP-004** SHOULD: Validation failures triggered by user input (user not found, event not found) SHOULD be logged at info level; system exceptions SHOULD be logged at error level.
- **R-DISCORD-HTTP-005** SHOULD: HTTPException error details SHOULD sanitize sensitive internal information while logging full exception details separately for debugging.

### Verify

```bash
# Verify Pydantic BaseModel usage for request validation
grep -r 'class.*BaseModel' monorepo/tlt/adapters/discord_adapter/ | grep -v '__pycache__'

# Verify HTTPException usage in endpoint handlers
grep -r 'raise HTTPException' monorepo/tlt/adapters/discord_adapter/rsvp.py | wc -l

# Verify message_id validation against bot.active_events
grep -r 'if.*not in bot.active_events' monorepo/tlt/adapters/discord_adapter/rsvp.py

# Verify defensive dictionary initialization patterns
grep -r "if.*not in event\['reactions'\]" monorepo/tlt/adapters/discord_adapter/rsvp.py
```

**Accept when:**
- All HTTP endpoint handlers validate message_id existence in bot.active_events before Discord API calls
- Request bodies use Pydantic BaseModel validation and handlers raise HTTPException for missing resources (404) and exceptions (500)
- Nested state structures are defensively initialized before mutation operations (if 'reactions' not in event: event['reactions'] = {})
- Validation failures are logged at appropriate levels (info for user-triggered, error for system exceptions)
- HTTPException responses do not expose sensitive internal error details to API consumers

<enforcement>
Clause Code MUST NOT skip or defer verification of these rules. All new HTTP endpoints in the Discord adapter must pass the verify commands and meet all accept criteria before merge.
</enforcement>