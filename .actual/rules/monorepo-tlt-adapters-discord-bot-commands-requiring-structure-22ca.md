# Use discord.py Modal UI Pattern for Structured Bot Input: Discord Bot Commands Requiring Structured Multi

These rules are ALWAYS ACTIVE for all Discord bot command implementations within the adapter layer requiring two or more structured input fields, including event creation, update, or configuration flows where users must provide multiple pieces of information.

### Rules

- **R-DISCORD-MODAL-001** MUST: Discord bot commands requiring structured multi-field user input MUST use `discord.ui.Modal` for form presentation rather than text parsing or alternative interaction patterns.
- **R-DISCORD-MODAL-002** MUST: All modal classes MUST inherit from `discord.ui.Modal` with a title parameter and define fields as `discord.ui.TextInput` class attributes.
- **R-DISCORD-MODAL-003** MUST: Modal `on_submit` methods MUST be async, receive a `discord.Interaction` parameter, and delegate to handler classes rather than implementing business logic inline.
- **R-DISCORD-MODAL-004** MUST: Modal instances MUST be instantiated with any required bot instance or context dependencies passed to `__init__`.
- **R-DISCORD-MODAL-005** MUST: Handler delegation pattern MUST pass the interaction object and field values to handler methods, allowing handlers to manage interaction responses (defer, respond, followup) according to processing requirements.
- **R-DISCORD-MODAL-006** SHOULD: Implement immediate interaction acknowledgment with deferred response pattern to handle long-running processing within Discord's timeout constraints.
- **R-DISCORD-MODAL-007** SHOULD: Offload long-running processing to background tasks to avoid exceeding Discord API interaction timeout windows.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact
find . -name 'requirements*.txt' -o -name 'pyproject.toml' -o -name 'Pipfile' -o -name 'poetry.lock' | head -5

# 2. Resolve the installed discord library version from lock artifact
grep -E '(discord|discord\.py)' $(find . -name '*.lock' -o -name 'requirements*.txt' | head -1) | head -3

# 3. Locate and inspect modal class definitions
find . -path './.actual' -prune -o -type f -name '*.py' -exec grep -l 'discord\.ui\.Modal\|from discord\.ui import Modal' {} \;

# 4. Verify modal classes inherit from discord.ui.Modal with title and TextInput fields
grep -A 10 'class.*Modal' $(find . -path './.actual' -prune -o -type f -name '*.py' -print | xargs grep -l 'discord\.ui\.Modal') | grep -E '(class|discord\.ui\.TextInput|title)'

# 5. Verify on_submit methods are async and receive discord.Interaction
grep -B 2 -A 5 'async def on_submit' $(find . -path './.actual' -prune -o -type f -name '*.py' -print | xargs grep -l 'discord\.ui\.Modal') | grep -E '(async def on_submit|discord\.Interaction)'

# 6. Locate and execute the project's test suite covering modal submission flows
find . -path './.actual' -prune -o -type f -name 'test_*.py' -o -name '*_test.py' -print | xargs grep -l 'modal\|Modal' | head -3

# 7. Verify test coverage for modal submission flows
grep -E '(on_submit|interaction|Modal)' $(find . -path './.actual' -prune -o -type f \( -name 'test_*.py' -o -name '*_test.py' \) -print | xargs grep -l 'modal\|Modal') | head -10
```

**Accept when:**
- All modal classes inherit from `discord.ui.Modal` with title parameter and define fields as `discord.ui.TextInput` class attributes
- Modal `on_submit` methods are async, receive `discord.Interaction` parameter, and delegate to handler classes rather than implementing business logic inline
- Test coverage verifies modal submission flows handle interaction responses within Discord timeout constraints
- No multi-field input commands use text parsing or alternative interaction patterns outside the documented exceptions
- Handler delegation pattern passes interaction object and field values to handler methods for response management

<enforcement>
Claude Code MUST NOT skip or defer verification. All modal implementations MUST be inspected to confirm compliance with R-DISCORD-MODAL-001 through R-DISCORD-MODAL-007. Violations MUST trigger code review feedback requiring refactoring to modal pattern for multi-field input scenarios within policy scope.
</enforcement>