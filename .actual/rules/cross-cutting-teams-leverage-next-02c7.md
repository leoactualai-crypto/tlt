# Adopt Next.js as Core Framework for Dashboard Application: Teams Leverage Next

These rules are ALWAYS ACTIVE for all Next.js applications and dashboard components within the monorepo, particularly those in dedicated directories such as `monorepo/tlt/nextjs_app/dashboard/` and similar frontend application structures.

### Rules

- **R-NEXTJS-001** SHOULD: Teams SHOULD leverage Next.js built-in features (routing, API routes, SSR/SSG) rather than introducing alternative solutions.
- **R-NEXTJS-002** MUST: Ensure next.config.ts exports a valid Next.js configuration object with appropriate TypeScript types imported from 'next'.
- **R-NEXTJS-003** SHOULD: Place Next.js applications in dedicated directories within the monorepo structure (e.g., apps/dashboard, packages/admin-ui).
- **R-NEXTJS-004** SHOULD: Configure monorepo build tools (Turborepo, Nx, etc.) to recognize Next.js build outputs and dependencies.
- **R-NEXTJS-005** SHOULD: Document Next.js version requirements and upgrade procedures in monorepo README or architecture documentation.
- **R-NEXTJS-006** SHOULD: Establish conventions for page routing, API routes, and shared component organization within Next.js applications.
- **R-NEXTJS-007** SHOULD: Implement abstraction layers for routing and data fetching to avoid tight coupling between application logic and Next.js APIs.
- **R-NEXTJS-008** SHOULD: Monitor build times in CI/CD and leverage Next.js incremental builds and caching to prevent performance degradation.

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
- Next.js imports are properly declared in configuration files

<enforcement>
Claude Code MUST NOT skip or defer verification. All Next.js applications must pass these checks before code review approval. Violations require Architecture Team review and explicit exception approval.
</enforcement>