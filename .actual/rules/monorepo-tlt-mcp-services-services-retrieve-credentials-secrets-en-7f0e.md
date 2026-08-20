# Environment Variable Retrieval for API Credentials: Services Retrieve Credentials Secrets Environment Variables

These rules are ALWAYS ACTIVE for all backend services that authenticate with external APIs and require secret credentials for third-party integrations.

### Rules

- **R-CRED-001** MUST: Services MUST retrieve API credentials and secrets from environment variables at runtime, not hardcode them in source files.
- **R-CRED-002** MUST: Validate that all required environment variables are present during service initialization, before attempting to use external APIs, to fail fast with clear error messages.
- **R-CRED-003** MUST: Document required environment variables in service README files, deployment guides, and infrastructure-as-code templates to ensure proper configuration across environments.
- **R-CRED-004** SHOULD: Implement a configuration validation module that centralizes environment variable retrieval and validation logic to ensure consistent error handling.
- **R-CRED-005** MUST: Implement logging filters to redact credential values in logs and error messages to prevent accidental exposure.
- **R-CRED-006** MUST: Handle credential retrieval errors without exposing credential values in error messages or stack traces.

### Verify

```bash
# Discover and execute the project's static analysis or linting configuration
# to detect hardcoded credential patterns in source files
find . -type f -name '.eslintrc*' -o -name 'pylintrc' -o -name '.flake8' -o -name 'sonar-project.properties' | head -1

# Discover and execute the project's test suite to verify that services
# handle missing environment variables gracefully with appropriate error messages
find . -type f \( -name 'package.json' -o -name 'pytest.ini' -o -name 'setup.py' -o -name 'Makefile' \) | head -1

# Inspect the repository to confirm no files contain credential values
grep -r "OPENAI_API_KEY\|api_key\|secret_key" --include="*.js" --include="*.ts" --include="*.py" --include="*.java" . 2>/dev/null | grep -v node_modules | grep -v '.git' | head -20

# Verify credential-related files are properly listed in version control ignore configurations
cat .gitignore .env.example 2>/dev/null | grep -E '\.env|secrets|credentials'
```

**Accept when:**
- Static analysis confirms no hardcoded credentials exist in source files
- Service initialization tests verify that missing required environment variables produce clear error messages before attempting API calls
- Version control ignore configurations prevent credential files from being committed
- All external API integrations retrieve credentials exclusively from environment variables
- Documentation clearly identifies all required environment variables with placeholder examples

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks, code review checklists, and CI pipeline static analysis MUST enforce these rules. No exceptions are permitted for production code. Test fixtures may use mock credentials clearly marked as non-functional examples. Documentation examples must use placeholder values with clear indication they are not real credentials.
</enforcement>