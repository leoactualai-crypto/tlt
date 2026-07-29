# Standardize Structured Logging with Named Loggers for Business Rule Traceability: Business Rule Execution

These rules are ALWAYS ACTIVE for all FastAPI router modules, domain validation modules, business logic processors, and service modules implementing business rules, domain validation, and API contracts within the monorepo/tlt domain.

### Rules

- **R-BRE-001** MUST: Business rule execution paths MUST emit structured log entries at appropriate levels (INFO for successful rule application, ERROR for validation failures, WARNING for degraded states) that include contextual identifiers (user_id, message_id, event_id, task_id).
- **R-BRE-002** MUST: Initialize module-scoped loggers at the top of each business rule module using `logger = logging.getLogger(__name__)` for standard library or `from loguru import logger` for loguru-based services, preserving module naming to maintain architectural boundary visibility.
- **R-BRE-003** MUST: Log business rule execution outcomes with contextual identifiers including entity IDs (user_id, message_id, event_id) for correlation in successful operations and validation failures.
- **R-BRE-004** MUST: Health check and monitoring endpoints MUST log status transitions with structured context including timestamp and service name, using consistent status values (healthy, degraded, unhealthy, warning) for operational alerting.
- **R-BRE-005** SHOULD: Configure log levels per environment and module (DEBUG for development, INFO for staging, WARNING/ERROR for production business rule paths) and use log sampling for high-frequency operations while preserving full logging for validation failures.

### Verify

```bash
# Verify logger initialization in business rule modules
grep -r 'logging.getLogger(__name__)\|from loguru import logger' monorepo/tlt --include='*.py' | grep -E '(reminder|experience|rsvp|monitor|photo_processor)\.py'

# Verify logging in domain validation modules with BaseModel
grep -r 'class.*BaseModel' monorepo/tlt --include='*.py' -A 20 | grep -B 20 'logger\.(info\|error\|warning\|debug)'

# Count logger usage in FastAPI router endpoints
grep -r '@router\.\(get\|post\|delete\)' monorepo/tlt --include='*.py' -A 30 | grep 'logger\.(info\|error)' | wc -l
```

**Accept when:**
- All modules defining Pydantic BaseModel schemas for business entities (ReminderCreate, ExperienceCreate, ReactionUpdate, TaskStatusResponse) initialize a module-scoped logger using `logging.getLogger(__name__)` or `loguru.logger`
- FastAPI router endpoint handlers implementing business rules emit structured log entries for execution outcomes (success, validation failure, not found, server error) with contextual identifiers (user_id, message_id, event_id)
- Health check and monitoring endpoints log status transitions (healthy, degraded, unhealthy, warning) with timestamp and service name at appropriate severity levels (ERROR for failures, WARNING for degraded states)
- Log entries include entity IDs and contextual information sufficient for correlation and debugging of business rule violations

<enforcement>
Claude Code MUST NOT skip or defer verification. Static analysis linting rules MUST detect missing logger initialization in modules with BaseModel or FastAPI router definitions. Code review MUST verify logging for business rule execution paths. CI pipeline MUST fail if business rule modules lack logger initialization. Violations block merge until remediated.
</enforcement>