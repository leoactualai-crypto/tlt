# Adopt PIL (Pillow) for Image Manipulation Operations: Before Implementing Any Pil Dependent Feature

These rules are ALWAYS ACTIVE for all Python services and utilities requiring programmatic image manipulation, including canvas rendering, composition operations, format conversion, and pixel-level image operations.

### Rules

- **R-20-001** MUST: Before implementing any PIL-dependent feature, developers MUST discover the project's dependency lock artifact, resolve the exact installed version of PIL, and verify API compatibility against that version's official documentation.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact, then verify PIL is declared and resolved to a specific version
grep -r "pillow\|PIL" pyproject.toml poetry.lock requirements.txt setup.py 2>/dev/null | head -20

# 2. Locate the project's test suite discovery mechanism and execute image processing test cases to verify PIL integration
find . -name "*test*image*" -o -name "*image*test*" | grep -E "\.py$" | head -10

# 3. Inspect import statements in identified files to confirm PIL and io module usage follows the BytesIO streaming pattern
grep -r "from PIL import\|import PIL\|from io import BytesIO\|import io" --include="*.py" | grep -E "(Image|ImageDraw|BytesIO)" | head -20
```

**Accept when:**
- Dependency verification confirms PIL is installed and locked to a specific version in the project's lock artifact
- All image processing test cases pass, demonstrating PIL APIs function correctly in the deployment environment
- Code inspection confirms Image.open, ImageDraw.Draw, and BytesIO integration patterns are present and functional
- API usage in implementation matches the exact version's official documentation

<enforcement>
Claude Code MUST NOT skip or defer verification. Before writing any PIL-dependent code, the lock-file version MUST be discovered and official documentation for that exact version MUST be consulted. Violation of R-20-001 blocks code generation.
</enforcement>