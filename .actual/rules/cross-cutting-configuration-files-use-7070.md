# Adopt Next.js as Core Framework for Dashboard Application: Configuration Files Use

These rules are ALWAYS ACTIVE for Next.js configuration files and dashboard application code within the monorepo.

### Rules

- **R-NEXTJS-001** MUST: Configuration files MUST use TypeScript (.ts extension) for type safety and IDE support

### Verify

```bash
# Verify Next.js config file exists
test -f monorepo/tlt/nextjs_app/dashboard/next.config.ts && echo 'Next.js config exists'

# Verify Next.js imports are present
grep -r "from 'next'" monorepo/tlt/nextjs_app/dashboard/next.config.ts

# Verify nextConfig export exists
grep -r "nextConfig" monorepo/tlt/nextjs_app/dashboard/next.config.ts
```

**Accept when:**
- next.config.ts file exists in the dashboard application directory
- Configuration file exports a nextConfig object that serves as the public API contract
- TypeScript is used for Next.js configuration files to ensure type safety

<enforcement>
Claude Code MUST NOT skip or defer verification. All Next.js configuration files must be validated against these rules before acceptance.
</enforcement>