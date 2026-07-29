# Standardize Structured Logging with Named Loggers for Business Rule Traceability: Domain Validation Failures

These rules are ALWAYS ACTIVE for all FastAPI router modules, domain validation modules, business logic processors, and service modules implementing business rules, domain validation, and API contracts within the monorepo/tlt domain.

### Rules

- **R-LOGGING-001** SHOULD: Domain validation failures (HTTPException, validation errors) SHOULD be logged with ERROR level before raising exceptions, including the validation rule violated and input data context.

### Verify

```bash
# Check for logger initialization in business rule modules
grep -r 'logging.getLogger(__name__)\|from loguru import logger' monorepo/tlt --include='*.py' | grep -E '(reminder|experience|rsvp|monitor|photo_processor)\.py'

# Verify logger usage alongside BaseModel definitions
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' -A 20 | grep -B 20 'logger\.(info\|error\|warning\|debug)'

# Count logger usage in FastAPI router endpoints
grep -r '@router\.\(get\|post\|delete\)' monorepo/tlt --include='*.py' -A 30 | grep 'logger\.(info\|error)' | wc -l
```

**Accept when:**
- All modules defining Pydantic BaseModel schemas for business entities (ReminderCreate, ExperienceCreate, ReactionUpdate, TaskStatusResponse) initialize a module-scoped logger using `logging.getLogger(__name__)` or `from loguru import logger`
- FastAPI router endpoint handlers implementing business rules emit structured log entries for execution outcomes (success, validation failure, not found, server error) with contextual identifiers (user_id, message_id, event_id)
- Health check and monitoring endpoints log status transitions (healthy, degraded, unhealthy, warning) with timestamp and service name at appropriate severity levels (ERROR for failures, WARNING for degraded states)
- Domain validation failures include the validation rule violated and relevant input data context in ERROR-level log entries before exceptions are raised

<enforcement>
Claude Code MUST NOT skip or defer verification of logger initialization and structured logging presence in business rule modules. All three verify commands MUST execute successfully before accepting changes to domain validation or business rule modules.
</enforcement>