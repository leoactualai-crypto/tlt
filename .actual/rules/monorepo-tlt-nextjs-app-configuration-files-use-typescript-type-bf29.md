# Adopt Next.js Framework with App Router for Dashboard Application: Configuration Files Use Typescript Type Safe

These rules are ALWAYS ACTIVE for all React components, pages, routing logic, server-side rendering, static generation requirements, and application-level layouts and metadata configuration within the dashboard application using Next.js App Router.

### Rules

- **R-NEXTJS-CONFIG-001** SHOULD: Configuration files SHOULD use TypeScript for type-safe framework configuration.

### Verify

```bash
# Discover the project's dependency manifest and lock file; verify the framework dependency is declared and resolved
grep -E '"next"|next:' package.json package-lock.json yarn.lock pnpm-lock.yaml 2>/dev/null | head -5

# Discover the project's build and development scripts; execute them to verify the application builds and runs successfully
cat package.json | grep -A 5 '"scripts"'

# Discover the app directory structure; verify root layout components exist and follow App Router conventions
find . -path './app/layout.tsx' -o -path './app/layout.ts' 2>/dev/null
```

**Accept when:**
- The framework dependency is present in the dependency manifest with a resolved version in the lock file
- The application builds without errors and serves successfully in development mode
- App directory structure contains valid layout components that render without runtime errors
- Configuration files (next.config.js, next.config.ts, tsconfig.json) use TypeScript or are typed via JSDoc

<enforcement>
Claude Code MUST NOT skip or defer verification. Build failures from missing framework dependencies block deployment. Code review identifies deviations from App Router conventions and requests corrections. Runtime errors from incorrect API usage are surfaced in development and testing environments.
</enforcement>