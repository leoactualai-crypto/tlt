# Use discord.py Modal UI Pattern for Structured Bot Input: Modal Submission Logic Delegate Separate Handler

These rules are ALWAYS ACTIVE for all Discord bot command implementations within the adapter layer requiring two or more structured input fields, including event creation, update, or configuration flows where users must provide multiple pieces of information.

### Rules

- **R-MODAL-001** SHOULD: Modal submission logic SHOULD delegate to separate handler classes rather than implementing business logic directly in the on_submit method.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact
find . -name 'requirements*.txt' -o -name 'pyproject.toml' -o -name 'Pipfile' -o -name 'poetry.lock' | head -5

# 2. Resolve the installed discord.py version from lock artifact
grep -i 'discord' $(find . -name '*.lock' -o -name 'requirements*.txt') | grep -v '#'

# 3. Locate modal class definitions in the codebase
grep -r 'class.*Modal.*discord\.ui\.Modal' --include='*.py' .

# 4. Verify modal on_submit methods delegate to handlers
grep -A 10 'async def on_submit' --include='*.py' -r . | grep -E '(handler|Handler|delegate)'

# 5. Inspect test coverage for modal submission flows
grep -r 'test.*modal\|modal.*test' --include='*.py' . | grep -i submit

# 6. Verify TextInput field definitions use discord.ui.TextInput
grep -r 'discord\.ui\.TextInput\|TextInput(' --include='*.py' .
```

**Accept when:**
- All modal classes inherit from `discord.ui.Modal` with title parameter and define fields as `discord.ui.TextInput` class attributes
- Modal `on_submit` methods are async, receive `discord.Interaction` parameter, and delegate to handler classes rather than implementing business logic inline
- Test coverage verifies modal submission flows handle interaction responses within Discord timeout constraints
- The installed discord.py version is verified against the project's lock artifact and matches the active environment
- Modal field definitions include appropriate configuration (style, default values, min_length constraints) per version-specific documentation

<enforcement>
Claude Code MUST NOT skip or defer verification. All modal implementations MUST be inspected to confirm handler delegation pattern is followed. Version-specific discord.py API usage MUST be verified against the exact resolved version before implementation.
</enforcement>