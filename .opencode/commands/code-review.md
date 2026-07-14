| ---
| description: "Run comprehensive code review with automated verification: typecheck, lint, security scan, and quality gate. Usage: /code-review [scope]"
| ---

# Code Review Command

Comprehensive code review of uncommitted changes with **automated verification**.

## Protocol

1. **Get changed files**: `git diff --name-only HEAD`
2. **Detect stack** from file extensions and project structure
3. **Run stack-appropriate verification** (see Verification by Stack)
4. **Review each changed file** for issues
5. **Generate report** with severity levels
6. **Block if CRITICAL/HIGH issues found**

## Verification by Stack

After reviewing, RUN the appropriate verification command:

| Stack | Verification Command | What It Catches |
|-------|--------------------|-----------------|
| Nuxt/Vue | `npx nuxi typecheck` | TS errors, type mismatches, unused imports |
| React/Next.js | `npx tsc --noEmit` | TS errors, type mismatches |
| Node.js | `npx tsc --noEmit` | TS errors, type mismatches |
| Flutter/Dart | `flutter analyze` | Lint errors, type issues, unused imports |
| Python | `ruff check .` | PEP 8, lint errors, unused imports |
| Go | `go vet ./...` | Suspicious constructs, bugs |
| Rust | `cargo check` + `cargo clippy` | Compilation + lint |

## Issue Severity Levels

### CRITICAL (Block commit)
- Hardcoded credentials, API keys, tokens
- SQL injection vulnerabilities (raw queries without parameterization)
- XSS vulnerabilities (unsanitized user output in HTML)
- Missing authentication on protected endpoints
- Path traversal risks (user-controlled file paths)
- Insecure deserialization
- Command injection (user input in exec/shell commands)

### HIGH (Must fix before commit)
- Functions > 50 lines without clear reason
- Files > 400 lines
- Nesting depth > 4 levels
- Missing error handling on I/O or network calls
- console.log / print statements in production code
- Unused imports or variables
- TODO/FIXME comments in committed code
- Missing input validation on API endpoints

### MEDIUM (Fix when possible)
- Mutation patterns where immutable would work
- Missing loading/error/empty states
- Hardcoded strings that should be constants/i18n keys
- Missing TypeScript types or `any` usage without justification
- Accessibility issues (missing alt text, poor contrast, missing labels)
- Missing tests for new/changed code

### LOW (Note, non-blocking)
- Minor formatting inconsistencies
- Variable names could be more descriptive
- Comments could be clearer
- Minor duplication

## Report Format

```markdown
## Code Review Report

### Scope
{files reviewed}

### Verification
- TypeCheck: ✅ PASS / ❌ FAIL
- Lint: ✅ PASS / ❌ FAIL / ⚠️ N/A
- Tests: ✅ PASS / ❌ FAIL / ⚠️ N/A

### Findings

| Severity | File | Line | Issue | Suggested Fix |
|----------|------|------|-------|---------------|
| HIGH | src/file.ts | 42 | Missing input validation | Add Zod schema |
| MEDIUM | src/file.ts | 15 | Unused import 'foo' | Remove import |
...

### Verdict
✅ APPROVED (no CRITICAL/HIGH issues)
❌ BLOCKED ({N} CRITICAL/HIGH issues found)
```

## Enforcement

- **BLOCK** commit if CRITICAL or HIGH issues found
- **FLAG** MEDIUM issues with suggested fixes
- **REPORT** LOW issues as notes
- Never approve code with security vulnerabilities
