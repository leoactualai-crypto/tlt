# Adopt Next.js as Core Framework for Dashboard Application: Next Configuration Export

These rules are ALWAYS ACTIVE for all Next.js configuration files and dashboard application code within the monorepo.

### Rules

- **R-NEXT-001** MUST: Next.js configuration MUST export a nextConfig object that defines the public contract for framework behavior.

### Verify

```bash
# Verify next.config.ts exists in dashboard directory
test -f monorepo/tlt/nextjs_app/dashboard/next.config.ts && echo 'Next.js config exists'

# Verify Next.js imports are present
grep -r "from 'next'" monorepo/tlt/nextjs_app/dashboard/next.config.ts

# Verify nextConfig export is present
grep -r "nextConfig" monorepo/tlt/nextjs_app/dashboard/next.config.ts
```

**Accept when:**
- next.config.ts file exists in the dashboard application directory
- Configuration file exports a nextConfig object that serves as the public API contract
- TypeScript is used for Next.js configuration files to ensure type safety

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verify commands must pass before accepting changes to Next.js configuration or dashboard application structure.
</enforcement>