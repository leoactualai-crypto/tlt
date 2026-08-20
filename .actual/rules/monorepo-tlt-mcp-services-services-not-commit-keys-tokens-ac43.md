# Environment Variable Retrieval for API Credentials: Services Not Commit Keys Tokens Passwords

These rules are ALWAYS ACTIVE for all backend services that authenticate with external APIs and all files in the codebase that may contain credentials or secrets.

### Rules

- **R-CRED-001** MUST_NOT: Services MUST NOT commit API keys, tokens, passwords, or other credentials to version control in any form.
- **R-CRED-002** MUST: Validate that all required environment variables are present during service initialization, before attempting to use external APIs, to fail fast with clear error messages.
- **R-CRED-003** MUST: Document required environment variables in service README files, deployment guides, and infrastructure-as-code templates to ensure proper configuration across environments.
- **R-CRED-004** SHOULD: Implement a configuration validation module that centralizes environment variable retrieval and validation logic to ensure consistent error handling.
- **R-CRED-005** SHOULD: Implement logging filters to redact credential values and handle credential retrieval errors without exposing values.

### Verify

```bash
# Discover and execute the project's static analysis or linting configuration
# to detect hardcoded credential patterns in source files
find . -name '.eslintrc*' -o -name 'pylintrc' -o -name '.flake8' -o -name 'bandit.yaml' | head -1

# Discover and execute the project's test suite to verify that services
# handle missing environment variables gracefully with appropriate error messages
find . -name 'package.json' -o -name 'pytest.ini' -o -name 'setup.py' | head -1

# Inspect the repository to confirm no files contain credential values
grep -r "OPENAI_API_KEY\|api_key\|API_KEY" --include="*.js" --include="*.ts" --include="*.py" --include="*.java" . 2>/dev/null | grep -v node_modules | grep -v '.env.example' | grep -v '.env.sample' || echo "No hardcoded credentials found"

# Verify credential-related files are properly listed in version control ignore configurations
grep -E '\.env|secrets|credentials|keys' .gitignore 2>/dev/null || echo "Check .gitignore for credential file patterns"
```

**Accept when:**
- Static analysis confirms no hardcoded credentials exist in source files
- Service initialization tests verify that missing required environment variables produce clear error messages before attempting API calls
- Version control ignore configurations prevent credential files from being committed
- All external API integrations retrieve credentials from environment variables at runtime
- Documentation clearly indicates which environment variables are required for each service

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks MUST block commits containing potential credentials. CI pipeline MUST fail builds that contain hardcoded credential patterns. Code review MUST require revision before merge if credentials are not properly externalized. No exceptions are permitted for production code.
</enforcement>