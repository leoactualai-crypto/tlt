# Use discord.py Modal UI Pattern for Structured Bot Input: Modal Classes Inherit Discord Provide Title

These rules are ALWAYS ACTIVE for all Discord bot command implementations within the adapter layer requiring two or more structured input fields, including event creation, update, and configuration flows.

### Rules

- **R-MODAL-001** MUST: Modal classes MUST inherit from discord.ui.Modal and provide a title parameter describing the form purpose.
- **R-MODAL-002** MUST: Modal on_submit methods MUST be async, receive discord.Interaction parameter, and delegate to handler classes rather than implementing business logic inline.
- **R-MODAL-003** MUST: TextInput field definitions MUST be declared as class attributes using discord.ui.TextInput with appropriate configuration (style, required, max_length, min_length).
- **R-MODAL-004** SHOULD: Modal classes should be instantiated with any required bot instance or context dependencies passed to __init__.
- **R-MODAL-005** SHOULD: Handler delegation pattern should pass interaction object and field values to handler methods, allowing handlers to manage interaction responses (defer, respond, followup) according to processing requirements.
- **R-MODAL-006** MUST: Modal submission flows MUST handle interaction responses within Discord timeout constraints using immediate interaction acknowledgment with deferred response pattern.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact
find . -name 'requirements*.txt' -o -name 'pyproject.toml' -o -name 'Pipfile' -o -name 'poetry.lock' | head -1

# 2. Resolve the installed discord library version from lock artifact
grep -E '(discord|discord\.py)' $(find . -name '*.lock' -o -name 'requirements*.txt' | head -1) | head -1

# 3. Locate modal class definitions in the codebase
find . -type f -name '*.py' -exec grep -l 'discord\.ui\.Modal\|from discord\.ui import Modal' {} \;

# 4. Verify modal inheritance and title parameter
grep -A 5 'class.*Modal' $(find . -type f -name '*.py' -exec grep -l 'discord\.ui\.Modal' {} \;) | grep -E '(class|title=)'

# 5. Verify TextInput field definitions
grep -B 2 -A 2 'discord\.ui\.TextInput' $(find . -type f -name '*.py' -exec grep -l 'discord\.ui\.Modal' {} \;)

# 6. Verify on_submit async implementation and handler delegation
grep -A 10 'async def on_submit' $(find . -type f -name '*.py' -exec grep -l 'discord\.ui\.Modal' {} \;)

# 7. Locate and execute test suite for modal submission flows
find . -type f -name 'test_*.py' -o -name '*_test.py' | xargs grep -l 'modal\|Modal' | head -1

# 8. Run tests to verify modal submission flows handle interactions correctly
python -m pytest $(find . -type f -name 'test_*.py' -o -name '*_test.py' | xargs grep -l 'modal\|Modal' | head -1) -v
```

**Accept when:**
- All modal classes inherit from discord.ui.Modal with title parameter and define fields as discord.ui.TextInput class attributes
- Modal on_submit methods are async, receive discord.Interaction parameter, and delegate to handler classes rather than implementing business logic inline
- TextInput field definitions include appropriate configuration options (style, required, max_length, min_length) per version-specific documentation
- Test coverage verifies modal submission flows handle interaction responses within Discord timeout constraints
- Modal classes are instantiated with required bot instance or context dependencies passed to __init__
- Handler delegation pattern passes interaction object and field values to handler methods for response management

<enforcement>
Claude Code MUST NOT skip or defer verification. All modal implementations MUST be inspected for R-MODAL-001 through R-MODAL-006 compliance before accepting code changes. Version-specific discord.py API documentation MUST be consulted per the LOCK-VERSION GROUNDING requirement before confirming API availability.
</enforcement>