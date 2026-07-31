# Adopt Next.js as Core Framework for Dashboard Application: Dashboard Applications Use

These rules are ALWAYS ACTIVE for all dashboard applications and Next.js configuration files within the monorepo.

### Rules

- **R-DASHBOARD-001** MUST: Dashboard applications MUST use Next.js as the core framework, configured via next.config.ts

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
Claude Code MUST NOT skip or defer verification. All dashboard applications MUST have a valid next.config.ts file with proper nextConfig export before code review approval.
</enforcement>