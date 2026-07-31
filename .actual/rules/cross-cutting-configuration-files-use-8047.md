# Adopt Next.js as Standard Framework for Public API Configuration: Configuration Files Use

These rules are ALWAYS ACTIVE for all Next.js applications and configuration files within the monorepo, particularly those in public API contexts and the dashboard application layer.

### Rules

- **R-NEXTJS-001** SHOULD: Configuration files SHOULD use TypeScript (.ts) rather than JavaScript (.js) for type safety.

### Verify

```bash
# Find all Next.js configuration files with nextConfig exports
find monorepo -name 'next.config.ts' -type f | xargs grep -l 'nextConfig'

# Verify Next.js is detected as a core library in public API contexts
grep -r "libs.core.detected.*next" monorepo/

# Find Next.js configuration files in application directories
find monorepo -path '*/nextjs_app/*/next.config.ts' -type f
```

**Accept when:**
- All Next.js applications contain a next.config.ts file with TypeScript typing
- The nextConfig export is present and properly typed in all Next.js configuration files
- Detection pipeline identifies Next.js as a core library (libs.core.detected='next') in public API contexts
- TypeScript compilation of Next.js configuration files succeeds in CI/CD pipeline

<enforcement>
Claude Code MUST NOT skip or defer verification of Next.js configuration file compliance. Configuration files MUST be TypeScript (.ts) files with proper nextConfig exports and type annotations.
</enforcement>