# Adopt PIL (Pillow) for Image Manipulation Operations: Components Requiring Programmatic Image Manipulation Operations

These rules are ALWAYS ACTIVE for all Python components requiring programmatic image manipulation operations, including canvas rendering, image overlay composition, format conversion, and pixel-level operations.

### Rules

- **R-PIL-001** MUST: All components requiring programmatic image manipulation operations (rendering, drawing, format conversion, pixel-level access) MUST use PIL (Pillow) as the image processing library.
- **R-PIL-002** MUST: When implementing BytesIO streaming patterns for image processing, ensure proper resource management by using context managers or explicit close operations to prevent memory leaks in long-running services.
- **R-PIL-003** SHOULD: Cache ImageDraw.Draw instances when performing multiple operations on the same image to avoid repeated context creation overhead.
- **R-PIL-004** SHOULD: Maintain clear separation of concerns when integrating PIL with OCR or other image analysis libraries: PIL handles manipulation and format operations, specialized libraries handle analysis.
- **R-PIL-005** MUST: Before writing code that uses PIL, execute lock-version grounding: (1) Find the dependency manifest in the repo, (2) Identify the build tool, (3) Inspect the repository lock artifact to determine exact resolved version, (4) Look up official documentation for that exact version, (5) Confirm every API, class, or function exists in that exact version's documentation before using it.
- **R-PIL-006** MUST: Implement image dimension validation before processing and establish maximum processable image size limits based on deployment environment constraints.
- **R-PIL-007** MUST: Monitor PIL security advisories and update promptly; implement input validation on image sources and consider sandboxing image processing operations for untrusted inputs.

### Verify

```bash
# Discover the project's dependency manifest and lock artifact
find . -name 'requirements*.txt' -o -name 'pyproject.toml' -o -name 'Pipfile' -o -name 'poetry.lock' | head -5

# Verify PIL is declared and resolved to a specific version
grep -r "pillow\|PIL" . --include='*.txt' --include='*.toml' --include='*.lock' | grep -v '.git'

# Locate and execute image processing test cases
find . -path '*/test*' -name '*image*' -type f | head -10

# Inspect import statements for PIL and io module usage
grep -r "from PIL import\|import PIL\|from io import BytesIO\|import io" . --include='*.py' | grep -v '.git' | head -20
```

**Accept when:**
- Dependency verification confirms PIL is installed and locked to a specific version in the project
- All image processing test cases pass, demonstrating PIL APIs function correctly in the deployment environment
- Code inspection confirms Image.open, ImageDraw.Draw, and BytesIO integration patterns are present and functional
- No alternative image processing libraries (OpenCV, imagemagick, etc.) are detected in image manipulation code paths

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules in this file are mandatory for code review and pull request acceptance.
</enforcement>