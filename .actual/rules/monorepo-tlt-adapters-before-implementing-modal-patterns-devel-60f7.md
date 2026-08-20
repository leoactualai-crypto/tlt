# Use discord.py Modal UI Pattern for Structured Bot Input: Before Implementing Modal Patterns Developers Discover

These rules are ALWAYS ACTIVE for all Discord bot command implementations within the adapter layer requiring two or more structured input fields, event creation/update flows, and scenarios where input validation and field constraints are required at the UI level.

### Rules

- **R-MODAL-001** MUST: Before implementing modal patterns, developers MUST discover the project's dependency lock artifact, resolve the exact installed version of the discord library, and verify API compatibility against that version's official documentation.
- **R-MODAL-002** MUST: All modal classes inherit from `discord.ui.Modal` with a title parameter and define fields as `discord.ui.TextInput` class attributes.
- **R-MODAL-003** MUST: Modal `on_submit` methods are async, receive `discord.Interaction` parameter, and delegate to handler classes rather than implementing business logic inline.
- **R-MODAL-004** MUST: Modal instances are instantiated with any required bot instance or context dependencies passed to `__init__`, as modals are created before user interaction occurs.
- **R-MODAL-005** SHOULD: Implement immediate interaction acknowledgment with deferred response pattern to handle Discord timeout constraints.
- **R-MODAL-006** SHOULD: Offload long-running processing to background tasks to avoid modal submission failures due to timeout windows.
- **R-MODAL-007** SHOULD: Document modal constraints (field count, character limits) in design phase and consider hybrid approaches when requirements exceed modal capabilities.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact
find . -name 'requirements*.txt' -o -name 'pyproject.toml' -o -name 'Pipfile' -o -name 'poetry.lock' | head -5

# 2. Resolve the installed discord library version from lock artifact
grep -E '^discord[=<>]|discord.*version' $(find . -name '*.lock' -o -name 'requirements*.txt' | head -1)

# 3. Verify modal class definitions inherit from discord.ui.Modal
grep -r 'class.*Modal.*discord\.ui\.Modal' --include='*.py' .

# 4. Verify TextInput field definitions
grep -r 'discord\.ui\.TextInput' --include='*.py' .

# 5. Verify async on_submit implementation
grep -r 'async def on_submit' --include='*.py' . | grep -v '__pycache__'

# 6. Locate and execute test suite covering modal submission flows
find . -name '*test*modal*' -o -name '*modal*test*' | grep -E '\.py$'

# 7. Run modal-related tests
python -m pytest -v $(find . -path '*test*' -name '*modal*' -type f)
```

**Accept when:**
- All modal classes inherit from `discord.ui.Modal` with title parameter and define fields as `discord.ui.TextInput` class attributes
- Modal `on_submit` methods are async, receive `discord.Interaction` parameter, and delegate to handler classes rather than implementing business logic inline
- Test coverage verifies modal submission flows handle interaction responses within Discord timeout constraints
- Dependency lock artifact is discovered and exact discord library version is verified against official documentation
- Modal implementations follow handler delegation pattern, passing interaction object and field values to handler methods
- TextInput field definitions use version-specific configuration options (style, default values, min_length) documented for the resolved discord library version

<enforcement>
Claude Code MUST NOT skip or defer verification of modal pattern compliance. All R-MODAL rules are mandatory before code review approval. Verification commands MUST be executed to confirm dependency resolution and modal implementation correctness.
</enforcement>