# Adopt PIL (Pillow) for Image Manipulation Operations: Drawing Operations Images Use Imagedraw Module

These rules are ALWAYS ACTIVE for all Python services and utilities requiring programmatic image manipulation, including canvas rendering, composition operations, image format conversion, and pixel-level image operations within the project.

### Rules

- **R-PIL-001** MUST: Drawing operations on images MUST use the ImageDraw module's Draw API to obtain a drawing context before performing shape, text, or overlay operations.

### Verify

```bash
# Discover the project's dependency manifest and lock artifact, then verify PIL is declared and resolved to a specific version
find . -name 'requirements*.txt' -o -name 'pyproject.toml' -o -name 'Pipfile' -o -name 'poetry.lock' | head -5

# Locate the project's test suite discovery mechanism and execute image processing test cases to verify PIL integration
find . -path '*/test*' -name '*image*' -type f | grep -E '\.(py)$' | head -10

# Inspect import statements in the identified files to confirm PIL and io module usage follows the BytesIO streaming pattern
grep -r "from PIL import\|import PIL\|from io import BytesIO\|import io" --include="*.py" | grep -E '(Image|ImageDraw|BytesIO)' | head -20
```

**Accept when:**
- Dependency verification confirms PIL (Pillow) is installed and locked to a specific version in the project's dependency manifest and lock artifact
- All image processing test cases pass, demonstrating PIL APIs function correctly in the deployment environment
- Code inspection confirms Image.open, ImageDraw.Draw, and BytesIO integration patterns are present and functional in image manipulation code
- No alternative image processing libraries (OpenCV, imagemagick, etc.) are used for drawing operations within scope

<enforcement>
Claude Code MUST NOT skip or defer verification. All image manipulation code using drawing operations MUST be reviewed against R-PIL-001 before acceptance.
</enforcement>