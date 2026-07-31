# Standardize Next.js Configuration as Public API Contract: Environment Specific Configuration

These rules are ALWAYS ACTIVE for Next.js configuration files (next.config.ts) in all Next.js applications within the monorepo.

### Rules

- **R-NEXTJS-CONFIG-001** SHOULD: Environment-specific configuration SHOULD be externalized through environment variables rather than hardcoded in the config file.

### Verify

```bash
# Verify configuration file exists
test -f monorepo/tlt/nextjs_app/dashboard/next.config.ts && echo 'Configuration file exists'

# Verify nextConfig export is present
grep -q 'nextConfig' monorepo/tlt/nextjs_app/dashboard/next.config.ts && echo 'nextConfig export found'

# Verify TypeScript configuration is valid
npx tsc --noEmit monorepo/tlt/nextjs_app/dashboard/next.config.ts && echo 'TypeScript configuration valid'
```

**Accept when:**
- next.config.ts exists in the Next.js application root directory
- Configuration file exports a valid nextConfig object conforming to Next.js schema
- TypeScript compilation succeeds without errors for the configuration file
- Environment-specific values are sourced from environment variables, not hardcoded
- No secrets or environment-specific configuration are hardcoded in the configuration file

<enforcement>
Claude Code MUST NOT skip or defer verification. Configuration files MUST be validated by TypeScript compiler and Next.js framework during build and startup phases. Violations block deployment in CI/CD pipeline.
</enforcement>