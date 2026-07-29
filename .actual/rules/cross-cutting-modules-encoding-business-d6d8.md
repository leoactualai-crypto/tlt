# Standardize Structured Logging with Named Loggers for Business Rule Traceability: Modules Encoding Business

These rules are ALWAYS ACTIVE for all modules encoding business rules through domain validation models (Pydantic BaseModel) or API contracts (FastAPI routers) within services implementing event management, reminders, experience tracking, photo processing, and monitoring.

### Rules

- **R-LOG-001** MUST: All modules encoding business rules through domain validation models (Pydantic BaseModel) or API contracts (FastAPI routers) MUST initialize a module-scoped logger using `logging.getLogger(__name__)` or equivalent structured logging framework (`loguru.logger`).

- **R-LOG-002** MUST: FastAPI router endpoint handlers implementing business rules MUST emit structured log entries for execution outcomes (success, validation failure, not found, server error) with contextual identifiers (user_id, message_id, event_id).

- **R-LOG-003** MUST: Health check and monitoring endpoints MUST log status transitions (healthy, degraded, unhealthy, warning) with timestamp and service name at appropriate severity levels (ERROR for failures, WARNING for degraded states).

- **R-LOG-004** SHOULD: Log business rule execution outcomes with contextual identifiers using patterns like `logger.info(f'User {user.name} reacted with {emoji} to event: {event["topic"]}')` for successful operations and `logger.error(f'Error handling reaction: {e}')` for failures.

- **R-LOG-005** SHOULD: Configure log levels per environment and module: DEBUG for development, INFO for staging, WARNING/ERROR for production business rule paths, with log sampling for high-frequency operations while preserving full logging for validation failures.

### Verify

```bash
# Check for logger initialization in business rule modules
grep -r 'logging.getLogger(__name__)\|from loguru import logger' monorepo/tlt --include='*.py' | grep -E '(reminder|experience|rsvp|monitor|photo_processor)\.py'

# Verify logger usage in BaseModel and router definitions
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' -A 20 | grep -B 20 'logger\.(info\|error\|warning\|debug)'

# Count structured logging in FastAPI endpoints
grep -r '@router\.\(get\|post\|delete\)' monorepo/tlt --include='*.py' -A 30 | grep 'logger\.(info\|error)' | wc -l
```

**Accept when:**
- All modules defining Pydantic BaseModel schemas for business entities (ReminderCreate, ExperienceCreate, ReactionUpdate, TaskStatusResponse) initialize a module-scoped logger using `logging.getLogger(__name__)` or `loguru.logger`
- FastAPI router endpoint handlers implementing business rules emit structured log entries for execution outcomes (success, validation failure, not found, server error) with contextual identifiers (user_id, message_id, event_id)
- Health check and monitoring endpoints log status transitions (healthy, degraded, unhealthy, warning) with timestamp and service name at appropriate severity levels (ERROR for failures, WARNING for degraded states)
- Module-scoped logger names preserve architectural boundary structure (adapters/discord_adapter, services/tlt_service, mcp_services/photo_vibe_check)

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis linting rules MUST detect missing logger initialization in modules with BaseModel or FastAPI router definitions. Code review MUST verify logging for business rule execution paths and validation failure handling. CI pipeline MUST fail if business rule modules lack logger initialization.
</enforcement>