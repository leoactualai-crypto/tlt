# Use discord.py Modal UI Pattern for Structured Bot Input: Modal Submission Handling Implemented Via Async

These rules are ALWAYS ACTIVE for all Discord bot command implementations within the adapter layer requiring two or more structured input fields, including event creation, update, and configuration flows.

### Rules

- **R-MODAL-001** MUST: Modal submission handling MUST be implemented via async on_submit method receiving discord.Interaction parameter.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact
find . -name 'requirements*.txt' -o -name 'pyproject.toml' -o -name 'Pipfile' -o -name 'poetry.lock' | head -5

# 2. Resolve the installed discord library version from lock artifact
grep -E '(discord|discord\.py)' $(find . -name '*.lock' -o -name 'requirements*.txt' | head -1) | head -3

# 3. Locate and execute the project's test suite covering modal submission flows
find . -path '*/test*' -name '*modal*' -type f | head -5

# 4. Inspect modal class definitions for inheritance and async on_submit
grep -r 'class.*Modal' --include='*.py' | grep -v '__pycache__'
grep -r 'async def on_submit' --include='*.py' | grep -v '__pycache__'

# 5. Verify discord.ui.Modal and discord.ui.TextInput usage
grep -r 'discord\.ui\.Modal\|discord\.ui\.TextInput' --include='*.py' | grep -v '__pycache__'
```

**Accept when:**
- All modal classes inherit from discord.ui.Modal with title parameter and define fields as discord.ui.TextInput class attributes
- Modal on_submit methods are async, receive discord.Interaction parameter, and delegate to handler classes rather than implementing business logic inline
- Test coverage verifies modal submission flows handle interaction responses within Discord timeout constraints
- The installed discord.py version matches the dependency manifest and supports the Modal and TextInput APIs being used

<enforcement>
Claude Code MUST NOT skip or defer verification. All modal implementations MUST be reviewed to confirm R-MODAL-001 compliance before acceptance.
</enforcement>