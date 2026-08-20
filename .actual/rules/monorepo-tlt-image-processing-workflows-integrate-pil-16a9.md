# Adopt PIL (Pillow) for Image Manipulation Operations: Image Processing Workflows Integrate Pil Ocr

These rules are ALWAYS ACTIVE for all Python services, utilities, and components requiring programmatic image manipulation, canvas rendering, format conversion, pixel-level operations, and image data processing as byte streams.

### Rules

- **R-PIL-001** MAY: Image processing workflows MAY integrate PIL with OCR libraries for text detection and extraction use cases, maintaining PIL as the image manipulation layer.
- **R-PIL-002** MUST: When implementing BytesIO streaming patterns, ensure proper resource management by using context managers or explicit close operations to prevent memory leaks in long-running services.
- **R-PIL-003** SHOULD: Cache ImageDraw.Draw instances when performing multiple operations on the same image to avoid repeated context creation overhead.
- **R-PIL-004** MUST: Maintain clear separation of concerns: PIL handles manipulation and format operations, specialized libraries handle analysis.
- **R-PIL-005** MUST: Enforce lock-file version resolution before implementation. Establish automated testing of image operations across target deployment environments.
- **R-PIL-006** MUST: Implement image dimension validation before processing and establish maximum processable image size limits based on deployment environment constraints.
- **R-PIL-007** MUST: Monitor PIL security advisories and update promptly. Implement input validation on image sources and add automated dependency vulnerability scanning to CI pipeline.

### Verify

```bash
# 1. Discover the project's dependency manifest and lock artifact
find . -name 'requirements*.txt' -o -name 'pyproject.toml' -o -name 'setup.py' -o -name 'Pipfile' -o -name 'poetry.lock' | head -5

# 2. Verify PIL is declared and resolved to a specific version
grep -i 'pillow\|PIL' $(find . -name 'requirements*.txt' -o -name 'pyproject.toml' -o -name 'poetry.lock' | head -1)

# 3. Locate and execute image processing test cases
find . -path '*/test*' -name '*image*' -type f | head -10

# 4. Inspect import statements for PIL and BytesIO usage patterns
grep -r 'from PIL import\|import PIL\|from io import BytesIO' --include='*.py' | grep -E '(Image|ImageDraw|BytesIO)' | head -20

# 5. Verify PIL APIs are used correctly
grep -r 'Image\.open\|ImageDraw\.Draw\|BytesIO' --include='*.py' | head -20
```

**Accept when:**
- Dependency verification confirms PIL (Pillow) is installed and locked to a specific version in the project's lock artifact
- All image processing test cases pass, demonstrating PIL APIs function correctly in the deployment environment
- Code inspection confirms Image.open, ImageDraw.Draw, and BytesIO integration patterns are present and functional
- Image processing operations use context managers or explicit resource cleanup for BytesIO streams
- No alternative image processing libraries (OpenCV, imagemagick, etc.) are introduced in scope without documented exception

<enforcement>
Claude Code MUST NOT skip or defer verification. All rules R-PIL-001 through R-PIL-007 are mandatory for image processing workflows. Dependency locking and security scanning (R-PIL-005, R-PIL-007) must be verified before code implementation.
</enforcement>