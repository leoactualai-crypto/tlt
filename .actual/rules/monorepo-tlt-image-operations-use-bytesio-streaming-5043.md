# Adopt PIL (Pillow) for Image Manipulation Operations: Image Operations Use Bytesio Streaming Pattern

These rules are ALWAYS ACTIVE for all Python services and utilities requiring programmatic image manipulation, including canvas rendering, composition operations, format conversion workflows, and pixel-level image operations within the scope of in-memory byte stream processing.

### Rules

- **R-PIL-001** SHOULD: Image I/O operations SHOULD use the BytesIO streaming pattern (Image.open with io.BytesIO) for in-memory processing when image data originates from or targets byte streams rather than filesystem paths.

### Verify

```bash
# Discover the project's dependency manifest and lock artifact
find . -name 'requirements*.txt' -o -name 'pyproject.toml' -o -name 'Pipfile' -o -name 'poetry.lock' | head -5

# Verify PIL is declared and resolved to a specific version
grep -i 'pillow\|PIL' $(find . -name 'requirements*.txt' -o -name 'pyproject.toml' -o -name 'poetry.lock' | head -1)

# Locate and execute image processing test cases
find . -path '*/test*' -name '*image*' -type f | head -10

# Inspect import statements for PIL and BytesIO usage patterns
grep -r 'from PIL import\|import PIL\|from io import BytesIO\|import io' --include='*.py' | grep -E '(Image|ImageDraw|BytesIO)' | head -20
```

**Accept when:**
- Dependency verification confirms PIL (Pillow) is installed and locked to a specific version in the project's dependency manifest and lock artifact
- All image processing test cases pass, demonstrating PIL APIs function correctly in the deployment environment
- Code inspection confirms Image.open, ImageDraw.Draw, and BytesIO integration patterns are present and functional in image processing modules
- No alternative image processing libraries (OpenCV, imagemagick, etc.) are introduced in-scope without documented exception and architecture review approval

<enforcement>
Claude Code MUST NOT skip or defer verification. Before approving image manipulation code, verify PIL dependency resolution, inspect BytesIO streaming patterns in implementation, and confirm test coverage for image operations.
</enforcement>