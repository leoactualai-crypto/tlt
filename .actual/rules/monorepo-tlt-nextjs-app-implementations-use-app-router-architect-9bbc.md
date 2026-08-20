# Adopt Next.js Framework with App Router for Dashboard Application: Implementations Use App Router Architecture Directory

These rules are ALWAYS ACTIVE for all React components, pages, layouts, and routing logic within the dashboard application using the Next.js App Router architecture.

### Rules

- **R-NEXTJS-001** MUST: Implementations MUST use the App Router architecture with app directory structure for layouts and pages.

### Verify

```bash
# Discover the project's dependency manifest and lock file; verify the framework dependency is declared and resolved
grep -E '"next"|next:' package.json package-lock.json yarn.lock pnpm-lock.yaml 2>/dev/null | head -5

# Discover the project's build and development scripts; execute them to verify the application builds and runs successfully
cat package.json | grep -A 5 '"scripts"'

# Discover the app directory structure; verify root layout components exist and follow App Router conventions
find . -path './app/layout.*' -o -path './app/page.*' 2>/dev/null | head -10
```

**Accept when:**
- The framework dependency is present in the dependency manifest with a resolved version in the lock file
- The application builds without errors and serves successfully in development mode
- App directory structure contains valid layout components that render without runtime errors
- Root layout components exist at `app/layout.tsx` (or equivalent) and define shared UI structure
- No legacy Pages Router structure (`pages/` directory) is used for new implementations

<enforcement>
Claude Code MUST NOT skip or defer verification. Build failures from missing framework dependencies block deployment. Code review identifies deviations from App Router conventions and requests corrections. Runtime errors from incorrect API usage are surfaced in development and testing environments.
</enforcement>