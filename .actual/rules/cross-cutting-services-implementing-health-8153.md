# Standardize Structured Logging with Named Loggers for Business Rule Traceability: Services Implementing Health

These rules are ALWAYS ACTIVE for all FastAPI-based services implementing business rules, domain validation, and API contracts that encode event management, reminders, experience tracking, photo processing, and monitoring logic.

### Rules

- **R-HEALTH-001** SHOULD: Services implementing health checks or monitoring endpoints SHOULD log status transitions (healthy, degraded, unhealthy, warning) with timestamp, service name, and diagnostic context to enable operational alerting.

### Verify

```bash
# Check for logger initialization in business rule modules
grep -r 'logging.getLogger(__name__)\|from loguru import logger' monorepo/tlt --include='*.py' | grep -E '(reminder|experience|rsvp|monitor|photo_processor)\.py'

# Verify logging in BaseModel validation modules
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' -A 20 | grep -B 20 'logger\.(info\|error\|warning\|debug)'

# Count logger usage in FastAPI router endpoints
grep -r '@router\.(get\|post\|delete)' monorepo/tlt --include='*.py' -A 30 | grep 'logger\.(info\|error)' | wc -l
```

**Accept when:**
- All modules defining Pydantic BaseModel schemas for business entities (ReminderCreate, ExperienceCreate, ReactionUpdate, TaskStatusResponse) initialize a module-scoped logger using `logging.getLogger(__name__)` or `from loguru import logger`
- FastAPI router endpoint handlers implementing business rules emit structured log entries for execution outcomes (success, validation failure, not found, server error) with contextual identifiers (user_id, message_id, event_id)
- Health check and monitoring endpoints log status transitions (healthy, degraded, unhealthy, warning) with timestamp and service name at appropriate severity levels (ERROR for failures, WARNING for degraded states)

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis linting rules MUST detect missing logger initialization in modules with BaseModel or FastAPI router definitions. Code review MUST verify logging for business rule execution paths. CI pipeline MUST fail if business rule modules lack logger initialization.
</enforcement>