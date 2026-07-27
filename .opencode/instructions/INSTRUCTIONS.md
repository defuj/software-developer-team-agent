# Everything Claude Code - OpenCode Instructions

This document consolidates the core rules and guidelines from the Claude Code configuration for use with OpenCode.

## Security Guidelines (CRITICAL)

### Mandatory Security Checks

Before ANY commit:

- [ ] No hardcoded secrets (API keys, passwords, tokens)
- [ ] All user inputs validated
- [ ] SQL injection prevention (parameterized queries)
- [ ] XSS prevention (sanitized HTML)
- [ ] CSRF protection enabled
- [ ] Authentication/authorization verified
- [ ] Rate limiting on all endpoints
- [ ] Error messages don't leak sensitive data

### Secret Management

```typescript
// NEVER: Hardcoded secrets
const apiKey = 'sk-proj-xxxxx';

// ALWAYS: Environment variables
const apiKey = process.env.OPENAI_API_KEY;

if (!apiKey) {
  throw new Error('OPENAI_API_KEY not configured');
}
```

### Security Response Protocol

If security issue found:

1. STOP immediately
2. Use **security-reviewer** agent
3. Fix CRITICAL issues before continuing
4. Rotate any exposed secrets
5. Review entire codebase for similar issues

---

## DELEGATION TIERS (Hemat Token, Gak Semua Full Pipeline)

Tidak semua UI task perlu 3-phase pipeline. Gunakan tier yang tepat:

| Tier          | Flow                                                            | Cocok Untuk                          |
| ------------- | --------------------------------------------------------------- | ------------------------------------ |
| **Fast** 🏃   | Leader → @frontend langsung                                     | Typo, spacing, warna, icon swap      |
| **Normal** ⚡ | Leader → @frontend + referensi DESIGN.md                        | Minor UI change (1-3 komponen)       |
| **Full** 🏗️   | Leader → @designer → Leader → @frontend → Leader → @designer QA | New feature, redesign, design system |

## SHARED ARTIFACTS PATTERN (Critical — Hemat Ratusan Token)

**⚠️ KENAPA INI PENTING**: Setiap subagent berjalan di **isolated context**. Mereka TIDAK otomatis tahu hasil subagent sebelumnya. Satu-satunya jembatan adalah file.

**Jangan forward data besar di pesan delegasi.** Gunakan file sebagai shared contract:

| Artifact                            | Isi                                                   | Ditulis Oleh                                       | Dibaca Oleh          |
| ----------------------------------- | ----------------------------------------------------- | -------------------------------------------------- | -------------------- |
| `DESIGN.md`                         | Design tokens, direction, component map               | @designer                                          | @frontend, @reviewer |
| `./specs/{feature}.md`              | Per-component specs (layout, states, variants, color) | @designer                                          | @frontend            |
| `./api-contract.md`                 | API contract (endpoints, request/response, auth)      | @leader / backend (node, ci3, laravel, java, go)   | @frontend, @reviewer |
| `./specs/implementation-summary.md` | Komponen dibuat, file paths, state handling           | @frontend (nuxt, react, flutter, android, angular) | @designer (QA)       |
| `./specs/review-report.md`          | Review findings + severity                            | @reviewer                                          | @leader              |
| `./specs/deployment-guide.md`       | Deployment config, env vars, build commands           | @devops                                            | @leader              |
| `./specs/seo-guide.md`              | Meta tags, structured data, sitemap changes           | @seo                                               | @frontend            |
| `prisma/schema.prisma`              | Model definitions final                               | @database                                          | @node-developer      |

**Aturan** (WAJIB dipatuhi semua agent):

1. **Producer** (designer, backend, database, dll): Setelah selesai, **TULIS output ke file** (jangan cuma return di message)
2. **Consumer** (frontend, reviewer, dll): Sebelum mulai, **BACA file** yang ditulis subagent sebelumnya
3. **Leader**: Setiap subagent selesai → **BACA file** → **VERIFIKASI file exist** → **BUAT ringkasan** → **SERTAKAN ringkasan + file reference** di delegasi berikutnya
4. **VERIFIKASI**: Jika file yang diharapkan tidak ditemukan, STOP & eskalasi — jangan lanjut delegasi dengan asumsi
5. Semua agent: **Jangan minta Leader forwarding data** — baca langsung dari file

### Prinsip: Deteksi Stuck, Bukan Batasi Iterasi

Task besar MEMBUTUHKAN banyak iterasi. Jangan batasi jumlah iterasi — batasi hanya ketika **tidak ada progress**.

### Stuck Detection: Same-Result Rule

Jika agent menerima **hasil/output yang SAMA PERSIS** dari subagent 2 kali berturut-turut:

1. STOP re-delegating (subagent stuck, ngulang hasil sama)
2. Dokumentasikan apa yang sama
3. Eskalasi ke user

### Kapan BUKAN Stuck (LANJUTKAN)

- Subagent menghasilkan kode berbeda setiap kali → ✅ progress
- Bug fix butuh banyak percobaan dengan pendekatan berbeda → ✅ progress
- Task besar dengan banyak delegasi → ✅ progress
- Pipeline UI bolak-balik Phase 2↔Phase 3 dengan perbaikan nyata → ✅ progress

---

## Quality Gate Checklist (ALL AGENTS — MANDATORY)

Before ANY agent marks work as `verified`, this checklist MUST pass:

### Mandatory Checks

- [ ] **Type check passes** (run `npx tsc --noEmit`, `dart analyze`, or equivalent)
- [ ] **No console.log/print() statements** in changed files
- [ ] **No unused imports** in changed files
- [ ] **Loading/error/empty states handled** for all data-dependent components
- [ ] **No hardcoded values** that should be config/props/constants
- [ ] **Immutability preserved** — no mutation of objects/arrays
- [ ] **Error handling in place** — try/catch or proper error boundaries
- [ ] **Verification status reported**: `verified` / `partially_verified` / `not_verified`

### Verification Commands by Stack

| Stack             | Minimum Verification            | Full Verification                                       |
| ----------------- | ------------------------------- | ------------------------------------------------------- |
| **Nuxt/Vue**      | `npx nuxi typecheck`            | `npx nuxi typecheck` + `npm run lint` + `npm run build` |
| **React/Next.js** | `npx tsc --noEmit`              | `npx tsc --noEmit` + `npm run lint` + `npm run build`   |
| **Flutter/Dart**  | `flutter analyze`               | `flutter analyze` + `flutter test`                      |
| **Node.js**       | `npx tsc --noEmit`              | `npx tsc --noEmit` + `npm run test`                     |
| **Angular**       | `ng lint`                       | `ng lint` + `ng test --watch=false` + `ng build`        |
| **Python**        | `ruff check .` or `pylint`      | `ruff check .` + `mypy .` + `pytest`                    |
| **Rust**          | `cargo check`                   | `cargo check` + `cargo clippy` + `cargo test`           |
| **Go**            | `go build ./...`                | `go build ./...` + `go vet ./...` + `go test ./...`     |
| **Java/Spring**   | `mvn compile` or `gradle build` | `mvn verify` or `gradle check`                          |

### Quality Enforcement Rules

1. **Must run minimum verification** before reporting `verified`
2. **If verification fails**, fix errors before reporting
3. **If verification cannot run** (missing config/tools), report as `partially_verified` with exact commands for manual verification
4. **Never skip verification** just because it's "a small change"
5. **Code review gate**: For medium+ changes, delegasikan ke `@code-reviewer` atau `@reviewer` sebelum final delivery

---

## UNIFIED CODING STANDARDS (ALL STACKS)

### Core Rules (Always Apply)

1. **Immutability**: ALWAYS create new objects, NEVER mutate
2. **Small files**: 200-400 lines typical, 800 max
3. **Small functions**: < 50 lines per function
4. **No deep nesting**: Max 4 levels of indentation
5. **Error handling**: Every operation that can fail MUST have error handling
6. **Input validation**: ALL user inputs validated (Zod, Joi, or equivalent)
7. **No hardcoded secrets**: Always use environment variables
8. **No console.log in production**: Search and remove before marking done

### Framework-Specific Rules

#### Nuxt/Vue

- Use `<script setup lang="ts">` always
- Use `useApi` composable for all API calls
- Prefer Nuxt UI components over custom HTML
- App directory structure (Nuxt 4)

#### React/Next.js

- Use TypeScript `interface` for props
- Server Components by default, `'use client'` only when needed
- Use shadcn/ui components over custom implementations
- App Router structure

#### Flutter/Dart

- Use `final` by default, `var` only when type is obvious
- Follow Clean Architecture (data/domain/presentation)
- Prefer Riverpod or Bloc for state management
- Use GoRouter for navigation

#### Node.js/Express

- `*.dto.ts`, `*.controller.ts`, `*.route.ts`, `*.middleware.ts` naming convention
- Use Prisma for database access
- JWT for authentication, stored in httpOnly cookies
- Zod or Joi for input validation

---

## Testing Requirements

### Minimum Test Coverage: 80%

Test Types (ALL required):

1. **Unit Tests** - Individual functions, utilities, components
2. **Integration Tests** - API endpoints, database operations
3. **E2E Tests** - Critical user flows (Playwright)

### Test-Driven Development

MANDATORY workflow:

1. Write test first (RED)
2. Run test - it should FAIL
3. Write minimal implementation (GREEN)
4. Run test - it should PASS
5. Refactor (IMPROVE)
6. Verify coverage (80%+)

### Troubleshooting Test Failures

1. Use **tdd-guide** agent
2. Check test isolation
3. Verify mocks are correct
4. Fix implementation, not tests (unless tests are wrong)

---

## Git Workflow

### Commit Message Format

```
<type>: <description>

<optional body>
```

Types: feat, fix, refactor, docs, test, chore, perf, ci

### Pull Request Workflow

When creating PRs:

1. Analyze full commit history (not just latest commit)
2. Use `git diff [base-branch]...HEAD` to see all changes
3. Draft comprehensive PR summary
4. Include test plan with TODOs
5. Push with `-u` flag if new branch

### Feature Implementation Workflow

1. **Plan First**
   - Use **planner** agent to create implementation plan
   - Identify dependencies and risks
   - Break down into phases

2. **TDD Approach**
   - Use **tdd-guide** agent
   - Write tests first (RED)
   - Implement to pass tests (GREEN)
   - Refactor (IMPROVE)
   - Verify 80%+ coverage

3. **Code Review**
   - Use **code-reviewer** agent immediately after writing code
   - Address CRITICAL and HIGH issues
   - Fix MEDIUM issues when possible

4. **Commit & Push**
   - Detailed commit messages
   - Follow conventional commits format

---

## Agent Orchestration

### Available Agents

| Agent                | Purpose                 | When to Use                   |
| -------------------- | ----------------------- | ----------------------------- |
| planner              | Implementation planning | Complex features, refactoring |
| architect            | System design           | Architectural decisions       |
| tdd-guide            | Test-driven development | New features, bug fixes       |
| code-reviewer        | Code review             | After writing code            |
| security-reviewer    | Security analysis       | Before commits                |
| build-error-resolver | Fix build errors        | When build fails              |
| e2e-runner           | E2E testing             | Critical user flows           |
| refactor-cleaner     | Dead code cleanup       | Code maintenance              |
| doc-updater          | Documentation           | Updating docs                 |
| go-reviewer          | Go code review          | Go projects                   |
| go-build-resolver    | Go build errors         | Go build failures             |
| database-reviewer    | Database optimization   | SQL, schema design            |

### Immediate Agent Usage

No user prompt needed:

1. Complex feature requests - Use **planner** agent
2. Code just written/modified - Use **code-reviewer** agent
3. Bug fix or new feature - Use **tdd-guide** agent
4. Architectural decision - Use **architect** agent

---

## Performance Optimization

### Model Selection Strategy

**Haiku** (90% of Sonnet capability, 3x cost savings):

- Lightweight agents with frequent invocation
- Pair programming and code generation
- Worker agents in multi-agent systems

**Sonnet** (Best coding model):

- Main development work
- Orchestrating multi-agent workflows
- Complex coding tasks

**Opus** (Deepest reasoning):

- Complex architectural decisions
- Maximum reasoning requirements
- Research and analysis tasks

### Context Window Management

Avoid last 20% of context window for:

- Large-scale refactoring
- Feature implementation spanning multiple files
- Debugging complex interactions

### Build Troubleshooting

If build fails:

1. Use **build-error-resolver** agent
2. Analyze error messages
3. Fix incrementally
4. Verify after each fix

---

## Common Patterns

### API Response Format

```typescript
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
  meta?: {
    total: number;
    page: number;
    limit: number;
  };
}
```

### Composable Pattern (Vue/Nuxt)

```typescript
export function useDebounce<T>(value: Ref<T>, delay: number): Readonly<Ref<T>> {
  const debouncedValue = ref<T>(value.value) as Ref<T>;

  watch(value, () => {
    const handler = setTimeout(() => {
      debouncedValue.value = value.value;
    }, delay);

    onScopeDispose(() => clearTimeout(handler));
  });

  return readonly(debouncedValue);
}
```

### Repository Pattern

```typescript
interface Repository<T> {
  findAll(filters?: Filters): Promise<T[]>;
  findById(id: string): Promise<T | null>;
  create(data: CreateDto): Promise<T>;
  update(id: string, data: UpdateDto): Promise<T>;
  delete(id: string): Promise<void>;
}
```

---

## OpenCode-Specific Notes

Since OpenCode does not support hooks, the following actions that were automated in Claude Code must be done manually:

### After Writing/Editing Code

- Run `prettier --write <file>` to format JS/TS files
- Run `npx tsc --noEmit` to check for TypeScript errors
- Check for console.log statements and remove them

### Before Committing

- Run security checks manually
- Verify no secrets in code
- Run full test suite

### Commands Available

Use these commands in OpenCode:

- `/plan` - Create implementation plan
- `/tdd` - Enforce TDD workflow
- `/code-review` - Review code changes
- `/security` - Run security review
- `/build-fix` - Fix build errors
- `/e2e` - Generate E2E tests
- `/refactor-clean` - Remove dead code
- `/orchestrate` - Multi-agent workflow

---

## Mobile Development

### Android (Kotlin / Jetpack Compose)

**Default Stack:**

- Kotlin, Jetpack Compose, Material Design 3, XML, Gradle KTS (version catalog)
- Hilt for DI, Room for local DB, Retrofit + OkHttp for networking
- Jetpack Navigation Compose, WorkManager

**Build Commands:**

```bash
./gradlew assembleDebug                  # Debug build
./gradlew assembleRelease                # Release APK
./gradlew bundleRelease                  # Play Store AAB
./gradlew test                           # Unit tests
./gradlew connectedAndroidTest           # Instrumented tests
```

**Play Store Publishing (via GPC):**

1. `gpc preflight <aab>` — Compliance check
2. `gpc publish --track internal` — Upload to internal track
3. `gpc release promote --from internal --to production` — Promote

### Flutter (Dart / Flutter SDK)

**Default Stack:**

- Dart, Flutter SDK, Material Design 3, Cupertino
- Bloc or Riverpod for state management
- GoRouter for routing, Dio for networking
- Hive/Isar/Drift for local storage

**Build Commands:**

```bash
flutter pub get                          # Install dependencies
flutter build apk                        # Android APK
flutter build appbundle                  # Android AAB
flutter build ios                        # iOS
flutter test                             # Run tests (unit + widget)
flutter test --coverage                  # With coverage
flutter analyze                          # Static analysis
```

**Testing Strategy:**

1. Unit tests — Domain logic, entities, usecases (flutter_test, mocktail)
2. Widget tests — UI components, states, interactions (WidgetTester)
3. Integration tests — Full user flows (integration_test package)
4. Golden tests — Visual regression (golden_toolkit, alchemist)

---

## Agentmemory: Persistent Cross-Session Memory

agentmemory provides persistent memory for all agents. It captures session history, saves decisions/insights, and injects relevant context from past sessions into the current session.

### Prerequisites

```bash
npm install -g @agentmemory/agentmemory   # Install globally
agentmemory                                # Start server on :3111
```

### MCP Tools (53 tools)

All agents have access to agentmemory MCP tools prefixed with `agentmemory_memory_`:

| Tool                       | Purpose                                                 |
| -------------------------- | ------------------------------------------------------- |
| `memory_save`              | Save insights, decisions, facts to long-term memory     |
| `memory_recall`            | Search past observations by keywords                    |
| `memory_smart_search`      | Hybrid semantic+keyword search for conceptual queries   |
| `memory_sessions`          | List recent sessions with status and observation counts |
| `memory_file_history`      | Get past observations about specific files              |
| `memory_lesson_save`       | Save a lesson learned with confidence scoring           |
| `memory_lesson_recall`     | Search lessons by query, sorted by confidence           |
| `memory_governance_delete` | Delete specific memories (requires confirmation)        |
| `memory_patterns`          | Detect recurring patterns across sessions               |
| `memory_consolidate`       | Run 4-tier memory consolidation pipeline                |

### Available Commands

- `/recall [query]` — Search past observations and lessons
- `/remember [text]` — Explicitly save an insight to long-term memory

### Auto-Capture Plugin

The `agentmemory-capture.ts` plugin (registered in `opencode.json`) captures 22 lifecycle events automatically:

- Session lifecycle: created, idle, compacted, updated, deleted, error
- Messages & prompts: user messages, assistant responses, removed messages
- Parts & steps: subagent starts, tool calls, reasoning, step-finish, patches, compaction events
- File enrichment: auto-injects file-specific context into system prompt
- Permissions: captures permission prompts and replies
- Tasks & commands: captures todo changes and command execution

### Skills

The `agentmemory` skill (`.opencode/skills/agentmemory/SKILL.md`) teaches agents when and how to use the memory tools effectively.

### Memory Workflow

The `agent-memory-workflow` skill (`.opencode/skills/agent-memory-workflow/SKILL.md`) provides disciplined protocols:

- **Session Start Ritual**: Mandatory memory recall before any productive work
- **Session End Ritual**: Structured save before session ends or compacts
- **Auto-Save Triggers**: When to save mid-session without waiting
- **Memory Audit**: Periodic hygiene and deduplication
- **Cross-Agent Memory**: Subagents don't write memory directly — IT Leader mediates

### Delegation Contracts

The `agent-delegation-contract` skill (`.opencode/skills/agent-delegation-contract/SKILL.md`) provides:

- **Standard Contract Template**: Structured brief with scope, input interface, output contract
- **Subagent Response Contract**: Mandatory result format including verification status
- **Chain vs Parallel Rules**: When to sequence vs parallelise delegations
- **Contract Violation Protocol**: How to handle out-of-scope work

### Progress Tracking

The `progress-tracking` skill (`.opencode/skills/progress-tracking/SKILL.md`) provides:

- **Task Lifecycle**: BACKLOG → TODO → IN_PROGRESS → REVIEW → DONE
- **Task ID Convention**: Hierarchical IDs (FE-001, BE-002) for cross-reference
- **Visual Progress Reports**: Tables with status emojis for user communication
- **Cross-Agent Protocol**: IT Leader maintains master list; subagents report results
- **Session Handoff**: How to recover task state after `/reset` or new session

---

## opencode-agent-kit Version Check

If `.opencode/.kit-version` exists, your agent toolkit has a recorded installed version.

**At session start**, check for updates:

1. Read `.opencode/.kit-version` to get the installed version
2. Run `npm view opencode-agent-kit version` to get the latest
3. If latest > installed, notify the user with the update command

This applies to IT-Leader (primary) and frontend/backend subagents when called directly.

### Global Install

The `global-install` skill (`.opencode/skills/global-install/SKILL.md`) documents the global installation approach. On macOS, OpenCode auto-discovers the global config at `~/.config/opencode/opencode.jsonc`. If the kit was installed globally, note that:

- Skills, prompts, commands, and instructions are stored flat in `~/.config/opencode/`
- `opencode.jsonc` is merged with your existing config (providers, MCP, etc.)
- No per-project `.opencode/` is needed — OpenCode reads the global config automatically
- To update: run `opencode-agent-kit global update`
- To install per-project: run `opencode-agent-kit init --local`

## UI Development Pipeline

This project uses a **3-phase UI Development Pipeline** for ALL UI-related tasks. Every agent involved in UI work must follow this protocol.

### The 3 Phases

```
PHASE 1 — DESIGN (@designer)
  │ Load `impeccable` skill (mandatory step 0)
  │ Use: /impeccable init, /impeccable shape, /impeccable critique
  │ Define: tokens, DESIGN.md, specs, states, a11y requirements
  │ Handoff to @frontend-nuxt / @frontend-react
  ▼
PHASE 2 — IMPLEMENT (@frontend)
  │ Implement from design specs
  │ MUST run Impeccable Polish Gate before done:
  │   /impeccable critique + /impeccable audit + /impeccable polish
  │ Extended: /impeccable harden, adapt, optimize, clarify, onboard, live
  │ Handoff to @designer for Phase 3 QA
  ▼
PHASE 3 — DESIGN QA (@designer)
  │ /impeccable critique on live result
  │ Verify: tokens, a11y, states, responsive, copy
  │ Report: PASS ✅ → done | FAIL ❌ → back to Phase 2
```

### Available Slash Commands

All Impeccable commands are available as OpenCode slash commands:

| Command                 | Shortcut     | Purpose                       |
| ----------------------- | ------------ | ----------------------------- |
| `/impeccable`           | —            | Main entry (23 subcommands)   |
| `/impeccable critique`  | `/critique`  | Full design review            |
| `/impeccable audit`     | `/audit`     | a11y, perf, responsive checks |
| `/impeccable polish`    | `/polish`    | Final quality pass            |
| `/impeccable craft`     | `/craft`     | End-to-end design + build     |
| `/impeccable shape`     | `/shape`     | UX planning                   |
| `/impeccable init`      | `/init`      | Setup context files           |
| `/impeccable live`      | `/live`      | Browser iteration             |
| `/impeccable document`  | `/document`  | Generate DESIGN.md            |
| `/impeccable extract`   | `/extract`   | Pull tokens                   |
| `/impeccable harden`    | `/harden`    | Edge cases                    |
| `/impeccable layout`    | `/layout`    | Spacing/rhythm                |
| `/impeccable typeset`   | `/typeset`   | Typography                    |
| `/impeccable colorize`  | `/colorize`  | Color                         |
| `/impeccable animate`   | `/animate`   | Motion                        |
| `/impeccable distill`   | `/distill`   | Simplify                      |
| `/impeccable clarify`   | `/clarify`   | UX copy                       |
| `/impeccable adapt`     | `/adapt`     | Responsive                    |
| `/impeccable bolder`    | `/bolder`    | Amplify                       |
| `/impeccable quieter`   | `/quieter`   | Tone down                     |
| `/impeccable delight`   | `/delight`   | Personality                   |
| `/impeccable overdrive` | `/overdrive` | Push limits                   |
| `/impeccable optimize`  | `/optimize`  | Performance                   |
| `/impeccable onboard`   | `/onboard`   | Empty states                  |
| `/impeccable hooks`     | `/hooks`     | Hook management               |

### Rules for All Agents

1. **UI task → delegate to @designer** (never do design work yourself)
2. **Design before code** — no frontend implementation without design specs
3. **Polish gate is mandatory** — frontend must run critique + audit + polish before done
4. **Design QA is mandatory** — designer must verify implementation before final
5. **All 23 commands available** — use the right command for the right task

## Success Metrics

You are successful when:

- All tests pass (80%+ coverage)
- No security vulnerabilities
- Code is readable and maintainable
- Performance is acceptable
- User requirements are met
