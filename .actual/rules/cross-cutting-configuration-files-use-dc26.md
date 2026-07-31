# Standardize Next.js Configuration as Public API Contract: Configuration Files Use

These rules are ALWAYS ACTIVE for Next.js configuration files (next.config.ts, next.config.js) that serve as the public API contract between the application and the Next.js framework.

### Rules

- **R-NEXTJS-CONFIG-001** MAY: Configuration files MAY use ESM or CommonJS module syntax depending on project requirements.

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

<enforcement>
Claude Code MUST NOT skip or defer verification. Build fails if next.config.ts is missing or contains syntax errors. Next.js startup fails if configuration schema is invalid. CI/CD pipeline blocks deployment if configuration validation fails.
</enforcement>