# Enforce Pydantic BaseModel for API Contract Validation in Public Interfaces: Services Not Accept

These rules are ALWAYS ACTIVE for all public API contracts, service boundaries, and domain validation models across MCP services (gateway, rsvp, event_manager, photo_vibe_check, guild_manager), agents (ambient_event_agent), adapters, and any code processing external input from Discord, CloudEvents, or HTTP requests.

### Rules

- **R-PYDANTIC-001** MUST_NOT: Services MUST NOT accept raw dictionaries or unvalidated JSON payloads directly into business logic without first parsing through a Pydantic BaseModel.

### Verify

```bash
# Count BaseModel class definitions across service, agent, and adapter modules
grep -r 'class.*BaseModel' monorepo/tlt/mcp_services monorepo/tlt/agents monorepo/tlt/adapters --include='*.py' | wc -l

# Verify request handlers use BaseModel validation
grep -r 'def.*request.*:' monorepo/tlt/mcp_services --include='*.py' -A 5 | grep -c 'BaseModel'

# Check all model classes inherit from BaseModel with explicit type annotations
python -c "import ast; import sys; [print(f'{n.name}: {any(b.id == \"BaseModel\" for b in n.bases if isinstance(b, ast.Name))}') for f in sys.argv[1:] for n in ast.walk(ast.parse(open(f).read())) if isinstance(n, ast.ClassDef)]" monorepo/tlt/mcp_services/*/models.py

# Detect raw dict usage in public service methods
grep -r 'def.*request.*:.*dict' monorepo/tlt/mcp_services --include='*.py' | grep -v 'BaseModel'
```

**Accept when:**
- All public API request/response models in mcp_services, agents, and adapters inherit from Pydantic BaseModel with explicit type annotations.
- Field constraints (ge, le, min_length, max_length, regex, default_factory) are applied to all fields with business rules or security requirements.
- No raw dictionary parameters are accepted in public service methods without prior Pydantic validation, verified by code review and linting.
- All external input handlers (Discord adapters, CloudEvents processors, HTTP endpoints) validate input through Pydantic models before passing to business logic.
- Authentication and authorization contexts (AuthContext, RBACRule) use Pydantic BaseModel with field validation.
- Inter-service contracts (MCPRequest, MCPResponse, AgentTask) are defined as Pydantic models with required field constraints.

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks, CI pipeline checks, and code review must confirm all public API boundaries use Pydantic BaseModel validation before accepting input into business logic.
</enforcement>