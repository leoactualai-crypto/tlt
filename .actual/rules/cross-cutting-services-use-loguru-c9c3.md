# Standardize Structured Logging with Named Loggers for Business Rule Traceability: Services Use Loguru

These rules are ALWAYS ACTIVE for all FastAPI router modules implementing business rule endpoints, domain validation modules defining Pydantic BaseModel schemas for business entities, business logic processors implementing rule evaluation, and service modules coordinating async business workflows within the monorepo/tlt domain.

### Rules

- **R-LOG-001** MUST: Initialize a module-scoped logger at the top of each business rule module using either `logger = logging.getLogger(__name__)` for standard library or `from loguru import logger` for loguru-based services, preserving module naming to maintain architectural boundary visibility.
- **R-LOG-002** MUST: Emit structured log entries for business rule execution outcomes (success, validation failure, not found, server error) with contextual identifiers (user_id, message_id, event_id) at appropriate severity levels.
- **R-LOG-003** MUST: Log validation failures and error conditions in business rule execution paths with sufficient context to enable debugging of specific rule violations.
- **R-LOG-004** SHOULD: Log health check and monitoring endpoint status transitions (healthy, degraded, unhealthy, warning) with timestamp and service name at appropriate severity levels (ERROR for failures, WARNING for degraded states).
- **R-LOG-005** MAY: Use loguru.logger as an alternative to logging.getLogger(__name__) for enhanced structured logging capabilities, provided module-level naming is preserved through configuration.
- **R-LOG-006** SHOULD: Configure log levels per environment and module (DEBUG for development, INFO for staging, WARNING/ERROR for production business rule paths) and use log sampling for high-frequency operations while preserving full logging for validation failures.

### Verify

```bash
# Verify logger initialization in business rule modules
grep -r 'logging.getLogger(__name__)\|from loguru import logger' monorepo/tlt --include='*.py' | grep -E '(reminder|experience|rsvp|monitor|photo_processor)\.py'

# Verify logging in modules with BaseModel schemas
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' -A 20 | grep -B 20 'logger\.(info\|error\|warning\|debug)'

# Count logger usage in FastAPI router endpoints
grep -r '@router\.\(get\|post\|delete\)' monorepo/tlt --include='*.py' -A 30 | grep 'logger\.(info\|error)' | wc -l
```

**Accept when:**
- All modules defining Pydantic BaseModel schemas for business entities (ReminderCreate, ExperienceCreate, ReactionUpdate, TaskStatusResponse) initialize a module-scoped logger using logging.getLogger(__name__) or loguru.logger
- FastAPI router endpoint handlers implementing business rules emit structured log entries for execution outcomes (success, validation failure, not found, server error) with contextual identifiers (user_id, message_id, event_id)
- Health check and monitoring endpoints log status transitions (healthy, degraded, unhealthy, warning) with timestamp and service name at appropriate severity levels (ERROR for failures, WARNING for degraded states)
- Module-scoped logger naming using __name__ is preserved to maintain architectural boundary structure and enable log filtering aligned with domain boundaries

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis linting rules MUST detect missing logger initialization in modules with BaseModel or FastAPI router definitions. Code review MUST verify logging for business rule execution paths and validation failure handling. CI pipeline MUST fail if business rule modules lack logger initialization. Violations MUST be addressed before merge.
</enforcement>