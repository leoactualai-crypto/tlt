# Enforce Pydantic BaseModel for API Contract Validation in Public Interfaces: Complex Validation Logic

These rules are ALWAYS ACTIVE for all public API contracts, service boundaries, and domain validation models across the codebase, including MCP services (gateway, rsvp, event_manager, photo_vibe_check, guild_manager), agents (ambient_event_agent), Discord adapters, authentication contexts, and external API request/response contracts.

### Rules

- **R-PYDANTIC-001** SHOULD: Complex validation logic SHOULD be implemented using Pydantic validators (@validator, @root_validator) rather than post-instantiation checks.

### Verify

```bash
# Count BaseModel classes across service boundaries
grep -r 'class.*BaseModel' monorepo/tlt/mcp_services monorepo/tlt/agents monorepo/tlt/adapters --include='*.py' | wc -l

# Verify request handlers use BaseModel
grep -r 'def.*request.*:' monorepo/tlt/mcp_services --include='*.py' -A 5 | grep -c 'BaseModel'

# Check BaseModel inheritance in model files
python -c "import ast; import sys; [print(f'{n.name}: {any(b.id == \"BaseModel\" for b in n.bases if isinstance(b, ast.Name))}') for f in sys.argv[1:] for n in ast.walk(ast.parse(open(f).read())) if isinstance(n, ast.ClassDef)]" monorepo/tlt/mcp_services/*/models.py

# Detect raw dict usage in public methods (security check)
grep -r 'def.*request.*:.*dict' monorepo/tlt/mcp_services --include='*.py'
```

**Accept when:**
- All public API request/response models in mcp_services, agents, and adapters inherit from Pydantic BaseModel with explicit type annotations.
- Field constraints (ge, le, min_length, max_length, regex, default_factory) are applied to all fields with business rules or security requirements.
- No raw dictionary parameters are accepted in public service methods without prior Pydantic validation, verified by code review and linting.
- Complex validation logic uses @validator or @root_validator decorators rather than post-instantiation conditional checks.

<enforcement>
Claude Code MUST NOT skip or defer verification. All new API endpoints and external input handlers MUST use Pydantic BaseModel with appropriate field constraints and validators. CI build fails if violations are detected.
</enforcement>