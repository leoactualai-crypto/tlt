# Use discord.py Modal UI Pattern for Structured Bot Input: Modal Field Values Accessed Via Value

These rules are ALWAYS ACTIVE for all Discord bot command implementations within the adapter layer requiring two or more structured input fields, including event creation, update, and configuration flows.

### Rules

- **R-MODAL-001** MAY: Modal field values MAY be accessed via the value attribute of TextInput instances within the on_submit callback.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact
find . -name 'requirements*.txt' -o -name 'pyproject.toml' -o -name 'Pipfile' -o -name 'poetry.lock' | head -5

# 2. Resolve the installed discord library version from lock artifact
grep -E '(discord|discord\.py)' $(find . -name '*.lock' -o -name 'requirements*.txt' | head -1) | head -3

# 3. Locate modal class definitions in the codebase
find . -type f -name '*.py' -exec grep -l 'discord\.ui\.Modal\|from discord\.ui import Modal' {} \;

# 4. Verify modal classes inherit from discord.ui.Modal and define TextInput fields
grep -A 10 'class.*Modal' $(find . -type f -name '*.py' -exec grep -l 'discord\.ui\.Modal' {} \;) | grep -E '(class|TextInput|on_submit)'

# 5. Verify on_submit methods are async and receive discord.Interaction
grep -A 5 'async def on_submit' $(find . -type f -name '*.py' -exec grep -l 'discord\.ui\.Modal' {} \;) | grep -E '(async|Interaction|self\..*\.value)'

# 6. Execute project test suite covering modal submission flows
find . -type f -name 'test*.py' -o -name '*test.py' | xargs grep -l 'modal\|Modal' | head -3
```

**Accept when:**
- All modal classes inherit from discord.ui.Modal with title parameter and define fields as discord.ui.TextInput class attributes
- Modal on_submit methods are async, receive discord.Interaction parameter, and delegate to handler classes rather than implementing business logic inline
- Field values are accessed via the .value attribute of TextInput instances within on_submit callbacks
- Test coverage verifies modal submission flows handle interaction responses within Discord timeout constraints
- The resolved discord.py version from the lock artifact supports the Modal and TextInput APIs being used

<enforcement>
Claude Code MUST NOT skip or defer verification. All modal implementations MUST be inspected to confirm inheritance, field definition, async on_submit signature, and value attribute access patterns before accepting code changes.
</enforcement>