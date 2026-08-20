# Adopt PIL (Pillow) for Image Manipulation Operations: Components Performing Format Conversion Encoding Integrate

These rules are ALWAYS ACTIVE for all Python services and utilities performing image manipulation, format conversion, encoding, canvas rendering, composition operations, pixel-level image operations including redaction and overlay, and components processing image data as byte streams in memory.

### Rules

- **R-PIL-001** SHOULD: Components performing format conversion or encoding SHOULD integrate PIL with the base64 module for web-compatible image serialization when transmitting images via text-based protocols.

### Verify

```bash
# Discover the project's dependency manifest and lock artifact, then verify PIL is declared and resolved to a specific version
find . -name 'requirements*.txt' -o -name 'pyproject.toml' -o -name 'Pipfile' -o -name 'poetry.lock' | head -5

# Locate the project's test suite discovery mechanism and execute image processing test cases to verify PIL integration
find . -path '*/test*' -name '*image*' -type f | head -10

# Inspect import statements in the identified files to confirm PIL and io module usage follows the BytesIO streaming pattern
grep -r "from PIL import\|import PIL\|from io import BytesIO\|import io" --include="*.py" | grep -E "(Image|ImageDraw|BytesIO)" | head -20
```

**Accept when:**
- Dependency verification confirms PIL is installed and locked to a specific version in the project's dependency manifest and lock artifact
- All image processing test cases pass, demonstrating PIL APIs function correctly in the deployment environment
- Code inspection confirms Image.open, ImageDraw.Draw, and BytesIO integration patterns are present and functional in image manipulation components

<enforcement>
Claude Code MUST NOT skip or defer verification. All three verification steps MUST complete successfully before accepting PIL integration in image manipulation components.
</enforcement>