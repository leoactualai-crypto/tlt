# Environment Variable Retrieval for API Credentials: Services Handle Missing Invalid Credentials Gracefully

These rules are ALWAYS ACTIVE for backend services that authenticate with external APIs and require secret credentials for third-party integrations.

### Rules

- **R-CRED-001** SHOULD: Services SHOULD handle missing or invalid credentials gracefully with clear error messages indicating which environment variable is required.

### Verify

```bash
# Discover and execute the project's static analysis or linting configuration to detect hardcoded credential patterns in source files
# (Exact command depends on project's build tool and linting setup — inspect repository for configuration)

# Discover and execute the project's test suite to verify that services handle missing environment variables gracefully with appropriate error messages
# (Exact command depends on project's test runner — inspect repository for test configuration)

# Inspect the repository to confirm no files contain credential values and that credential-related files are properly listed in version control ignore configurations
git check-ignore -v .env .env.local secrets.* || echo "Checking ignore patterns..."
```

**Accept when:**
- Static analysis confirms no hardcoded credentials exist in source files
- Service initialization tests verify that missing required environment variables produce clear error messages before attempting API calls
- Version control ignore configurations prevent credential files from being committed

<enforcement>
Claude Code MUST NOT skip or defer verification. Pre-commit hooks, CI pipeline static analysis, and code review checklist items enforce this rule. No exceptions are permitted for production code.
</enforcement>