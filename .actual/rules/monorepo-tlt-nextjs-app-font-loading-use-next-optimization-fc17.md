# Adopt Next.js Framework with App Router for Dashboard Application: Font Loading Use Next Optimization Rather

These rules are ALWAYS ACTIVE for all React components, pages, and layout files within the dashboard application using Next.js App Router.

### Rules

- **R-NEXTJS-FONT-001** SHOULD: Font loading SHOULD use Next.js font optimization APIs rather than manual font loading.

### Verify

```bash
# Discover the project's dependency manifest and lock file; verify the framework dependency is declared and resolved
grep -E '"next"|next:' package.json
cat package-lock.json | grep -A 5 '"next"' || cat yarn.lock | grep -A 5 'next@' || cat pnpm-lock.yaml | grep -A 5 'next:'

# Discover the project's build and development scripts; execute them to verify the application builds and runs successfully
cat package.json | grep -A 10 '"scripts"'
npm run build || yarn build || pnpm build

# Discover the app directory structure; verify root layout components exist and follow App Router conventions
find . -path './app/layout.tsx' -o -path './app/layout.jsx' -o -path './app/layout.js'
grep -r 'next/font' app/ || echo "Font optimization imports not yet present"
```

**Accept when:**
- The framework dependency is present in the dependency manifest with a resolved version in the lock file
- The application builds without errors and serves successfully in development mode
- App directory structure contains valid layout components that render without runtime errors
- Font loading in layout and page components uses Next.js font optimization APIs (e.g., `next/font/google`, `next/font/local`) rather than manual `@font-face` or external stylesheet imports

<enforcement>
Claude Code MUST NOT skip or defer verification. All font loading implementations MUST be reviewed against this rule before acceptance.
</enforcement>