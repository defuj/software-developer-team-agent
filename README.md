# Frontend Developer Agent (Nuxt) - Setup Guide

Panduan ini menjelaskan cara menggunakan agent di repo ini, termasuk skill yang perlu diinstall agar agent berjalan optimal.

## Gambaran Singkat

Repo ini berisi konfigurasi OpenCode untuk agent frontend & backend khusus Nuxt/Vue dan Node.js:

- Agent config: `.opencode/config.json`
- Agent prompts:
  - `.opencode/agents/nuxt-frontend-developer.md` — Nuxt frontend developer
  - `.opencode/agents/node-backend-developer.md` — Node.js backend developer
  - `.opencode/agents/nuxt-frontend-developer-mentor.md` — Nuxt mentor untuk junior/intermediate
- Dokumentasi internal: `.opencode/agent-docs/`
  - Frontend Nuxt: `.opencode/agent-docs/frontend/nuxt/`
  - Backend Node: `.opencode/agent-docs/backend/node/`
- Skill lokal: `.opencode/skills/` (33 skill tersinkron)
- Contexts: `.opencode/contexts/` (dev, research, review)

Agent didesain untuk:

- Nuxt 4 + Nuxt UI + Vue 3 Composition API + TypeScript
- Node.js + Express 5 + Prisma + PostgreSQL (backend agent)
- Workflow operasional tim (scope-safe, verification status, commit/PR policy)
- Mentoring terstruktur 30 hari untuk transisi ke stack Nuxt modern

## Prasyarat

- OpenCode CLI sudah terpasang
- Akses ke repository ini
- Node.js + npm/pnpm/yarn/bun sesuai kebutuhan proyek

## Agent yang Tersedia

Repo ini menyediakan 3 agent dengan arsitektur **Leader → Subagent**:

| Agent | File | Mode | Tujuan |
|-------|------|------|--------|
| **IT Leader** | `it-leader.md` | **primary** | Analisis requirement, arsitektur, pembagian tugas, delegasi, integrasi |
| Nuxt Frontend Developer | `nuxt-frontend-developer.md` | subagent | Implementasi frontend (komponen, halaman, composable, E2E) |
| Node Backend Developer | `node-backend-developer.md` | subagent | Implementasi backend (API, DTO, controller, database, auth) |

### Cara Kerja

1. User memberikan requirement ke **IT Leader** (`@leader`)
2. IT Leader menganalisis, merancang arsitektur, dan memecah menjadi task
3. IT Leader mendelegasikan task ke subagent yang sesuai (`@frontend` atau `@backend`)
4. IT Leader mengintegrasikan hasil dari subagent dan melaporkan ke user

Untuk task kecil yang langsung tahu subagent-nya, bisa langsung mention subagent:

```text
@frontend Tambahkan UButton "Simpan" di ProfileHeader.vue.
@backend Add endpoint POST /api/markets dengan DTO validation.
```

## Skill yang Digunakan Agent

Skill tersimpan di `.opencode/skills/` (lokal dalam repo), jadi developer lain tidak perlu mencari skill satu per satu.

### Skill minimal yang wajib tersedia

1. `coding-standards`
2. `frontend-patterns`
3. `frontend-design`
4. `web-design-guidelines`
5. `nuxt-ui`
6. `security-review`
7. `tdd-workflow`

### Skill tambahan (opsional, tapi direkomendasikan)

1. `building-components` (untuk pembuatan komponen reusable skala besar)
2. `vercel-composition-patterns` (untuk refactor komponen kompleks)

### Skill backend (untuk node-backend-developer agent)

1. `backend-patterns`
2. `postgres-patterns`

### Skill yang tidak perlu untuk operasional agent frontend ini

Untuk proyek Nuxt frontend ini, skill backend/lintas bahasa di bawah tidak diperlukan agar agent frontend berjalan optimal:

- `springboot-*`, `java-*`, `jpa-patterns`
- `django-*`
- `golang-*`
- `python-*`
- `clickhouse-io`

Catatan: skill tersebut boleh tetap disimpan jika tim memang butuh multi-stack, tapi tidak wajib untuk agent frontend ini.

### Skill lain yang tersedia di repo

- `configure-ecc` — installer untuk Everything Claude Code
- `continuous-learning` / `continuous-learning-v2` — pattern learning dari session
- `eval-harness` — formal evaluation framework
- `iterative-retrieval` — context retrieval pattern
- `strategic-compact` — manual context compaction
- `verification-loop` — verification cycle untuk agent
- `nutrient-document-processing` — document processing API
- `project-guidelines-example` — contoh panduan proyek

## Lokasi Skill

OpenCode membaca skill dari:

- `~/.opencode/skills/`
- `~/.agents/skills/`
- `.opencode/skills/` (lokal di repo ini)

Verifikasi cepat:

```bash
ls ~/.opencode/skills
ls ~/.agents/skills
ls .opencode/skills
```

Pastikan semua skill wajib tersedia.

## Cara Install Skill dari Folder `.opencode/skills/` Repo

Jika skill belum ada di mesin developer, copy dari repo ini ke direktori skill lokal:

```bash
mkdir -p ~/.opencode/skills
cp -R ./.opencode/skills/coding-standards ~/.opencode/skills/
cp -R ./.opencode/skills/frontend-patterns ~/.opencode/skills/
cp -R ./.opencode/skills/frontend-design ~/.opencode/skills/
cp -R ./.opencode/skills/web-design-guidelines ~/.opencode/skills/
cp -R ./.opencode/skills/nuxt-ui ~/.opencode/skills/
cp -R ./.opencode/skills/security-review ~/.opencode/skills/
cp -R ./.opencode/skills/tdd-workflow ~/.opencode/skills/
```

Opsional:

```bash
cp -R ./.opencode/skills/building-components ~/.opencode/skills/
cp -R ./.opencode/skills/vercel-composition-patterns ~/.opencode/skills/
```

## MCP yang Digunakan Agent

Dari `.opencode/config.json`, agent memakai MCP berikut:

| MCP | Type | Status | Deskripsi |
|-----|------|--------|-----------|
| `nuxt` | remote | enabled | Dokumentasi Nuxt, blog, deployment guide |
| `nuxt-ui` | remote | enabled | Dokumentasi & contoh komponen Nuxt UI |
| `playwright` | stdio | enabled | Browser automation & E2E testing |
| `figma` | stdio | disabled | Akses Figma design file (opsional) |

Jika ingin pakai Figma MCP, set env var:

```bash
export FIGMA_ACCESS_TOKEN="your-token"
```

## Cara Menggunakan Folder Pendukung

### `.opencode/contexts/`

Berisi konteks reusable proyek:

- `dev.md` — konteks development
- `research.md` — konteks riset
- `review.md` — konteks code review

Rujuk konteks ini saat membuat prompt supaya agent tidak keluar jalur. Update saat ada perubahan requirement produk/arsitektur.

Instal ke lokal jika ingin reuse context lintas repo:

```bash
mkdir -p ~/.opencode/contexts
cp -R ./.opencode/contexts/* ~/.opencode/contexts/
```

### `.opencode/commands/`, `.opencode/rules/`, `.opencode/hooks/`

Folder-folder ini ada di dalam `.opencode/` dan berisi:

- **`commands/`** — Command/prompt siap pakai (slash command internal tim)
- **`rules/`** — Aturan operasional/coding yang harus dipatuhi
- **`hooks/`** — Automasi lifecycle (pre-task/post-task, validasi)

Folder-folder ini **bukan** otomatis aktif. Developer perlu copy/sinkron ke environment lokal masing-masing jika diperlukan.

Instal commands:

```bash
mkdir -p ~/.opencode/commands
cp -R ./.opencode/commands/* ~/.opencode/commands/
```

Instal rules (ikuti README di `rules/` — copy per direktori, jangan di-flatten).

Hooks: aktifkan hanya jika environment lokal mendukung hook dependencies-nya. File `hooks/hooks.json` memanggil script via `CLAUDE_PLUGIN_ROOT`.

### Praktik Tim yang Disarankan

- Gunakan `commands/` untuk pekerjaan rutin.
- Gunakan `rules/` sebagai standar wajib saat implementasi dan review.
- Gunakan `contexts/` untuk prompt yang lebih presisi.
- Perlakukan `hooks/` sebagai konfigurasi bersama tim (perubahan perlu sinkronisasi).

## Cara Pakai

### Workflow Utama (via IT Leader)

IT Leader adalah **primary agent** — otomatis aktif saat session dimulai. Berikan requirement ke IT Leader:

```text
Buat fitur marketplace dengan halaman listing, detail, dan form create.
Backend API untuk CRUD markets dengan pagination dan filter.
```

IT Leader akan:
1. Analisis requirement dan definisikan scope
2. Rancang arsitektur (data flow, API contract, component structure)
3. Pecah menjadi task dan delegasikan ke `@frontend` / `@backend`
4. Integrasikan hasil dan laporkan status

### Direct Call ke Subagent (untuk task kecil)

Untuk task kecil yang sudah jelas subagent-nya, bisa langsung mention:

```text
@frontend Tambahkan UButton "Simpan" di app/components/profile/ProfileHeader.vue.
Task tiny, minimal diff, jangan ubah file lain.
```

Atau untuk task normal:

```text
@frontend Implementasikan filter status di halaman markets.
Gunakan pola useApi yang sudah ada dan laporkan verification status.
```

Untuk backend:

```text
@backend Add endpoint POST /api/markets dengan DTO validation.
```

## Standar Output Agent

Agent ini dikonfigurasi untuk selalu melaporkan:

- perubahan yang dibuat
- file yang disentuh
- status verifikasi: `verified` / `partially_verified` / `not_verified`
- command manual jika verifikasi penuh tidak bisa dijalankan

## Kebijakan Operasional Penting

- Tidak commit kecuali diminta user
- Tidak buat PR kecuali diminta user
- Tidak push kecuali diminta user
- Tidak menyentuh file di luar scope request

Detail SOP tim ada di: `.opencode/agent-docs/frontend/nuxt/TEAM_OPERATING_GUIDE.md`

## Dokumentasi Lengkap

Dokumentasi agent tersedia di `.opencode/agent-docs/frontend/nuxt/`:

| Dokumen | Isi |
|---------|-----|
| `INDEX.md` | Navigasi lengkap semua dokumentasi |
| `README.md` | User guide lengkap |
| `QUICK_START.md` | Mulai dalam 5 menit |
| `EXAMPLES.md` | 50+ contoh praktis |
| `API_PATTERNS.md` | Panduan useApi composable |
| `WORKFLOWS.md` | 8 workflow detail |
| `CHEATSHEET.md` | Quick reference |
| `MCP_GUIDE.md` | Panduan integrasi MCP |
| `TESTING_GUIDE.md` | Testing & validasi |
| `TEAM_OPERATING_GUIDE.md` | SOP tim |
| `SUMMARY.md` | Ringkasan instalasi |
| `COMPLETION_REPORT.md` | Laporan penyelesaian |
| `README_AGENTS.md` | Overview agent |
| `README_DOCS.md` | Overview dokumentasi |

### Dokumentasi Mentor

| Dokumen | Isi |
|---------|-----|
| `MENTOR_CURRICULUM_30_DAYS.md` | Roadmap belajar 30 hari |
| `MENTOR_CURRICULUM_CHECKLIST.md` | Checklist harian |
| `MENTOR_WEEKLY_ASSIGNMENTS.md` | Tugas mingguan + rubrik |

### Dokumentasi Backend

| Dokumen | Isi |
|---------|-----|
| `backend/README.md` | Overview backend |
| `backend/node/BACKEND_QUICK_START.md` | Quick start backend |
| `backend/node/BACKEND_PATTERNS.md` | Pola backend |

## Referensi Cepat

- Config: `.opencode/config.json`
- Prompt IT Leader (primary): `.opencode/agents/it-leader.md`
- Prompt frontend subagent: `.opencode/agents/nuxt-frontend-developer.md`
- Prompt backend subagent: `.opencode/agents/node-backend-developer.md`
- Dokumentasi utama: `.opencode/agent-docs/frontend/nuxt/README.md`
- Quick start: `.opencode/agent-docs/frontend/nuxt/QUICK_START.md`
- Index dokumentasi: `.opencode/agent-docs/frontend/nuxt/INDEX.md`

## Troubleshooting Instalasi Skill

### 1) Skill tidak muncul saat dicek

**Gejala:** nama skill tidak terlihat saat `ls ~/.opencode/skills` atau `ls ~/.agents/skills`.

**Langkah perbaikan:**

1. Pastikan path home benar:

```bash
echo $HOME
```

2. Cek direktori skill:

```bash
ls ~/.opencode/skills
ls ~/.agents/skills
ls .opencode/skills
```

3. Jika belum ada, install/copy skill ke salah satu direktori tersebut.

### 2) Skill ada, tapi agent tidak memakainya

**Gejala:** agent tidak memuat skill yang seharusnya dipakai.

**Langkah perbaikan:**

1. Panggil skill secara eksplisit di prompt:

```text
@frontend Load skill `nuxt-ui` lalu implementasikan form ini.
```

2. Mulai session baru OpenCode setelah update skill/konfigurasi.
3. Pastikan nama skill persis sama dengan folder skill.

### 3) Permission command diblokir

**Gejala:** agent tidak bisa menjalankan command tertentu (test/build/lint).

**Langkah perbaikan:**

1. Cek policy di `.opencode/config.json` pada bagian `agent.frontend.permission`.
2. Izinkan command yang dibutuhkan (mis. `pnpm *`, `npm *`, atau command spesifik).
3. Jalankan ulang task; agent akan melaporkan `verification status` sesuai izin yang tersedia.

### 4) MCP tidak tersedia atau gagal dipakai

**Gejala:** lookup Nuxt/Nuxt UI docs tidak berjalan.

**Langkah perbaikan:**

1. Cek `mcp` di `.opencode/config.json` pastikan `enabled: true` untuk `nuxt` dan `nuxt-ui`.
2. Pastikan koneksi internet aktif (karena MCP Nuxt/Nuxt UI remote).
3. Untuk Figma MCP, pastikan token terpasang:

```bash
export FIGMA_ACCESS_TOKEN="your-token"
```

### 5) Output agent terlalu verbose atau terlalu luas

**Gejala:** perubahan melebar atau penjelasan terlalu panjang.

**Langkah perbaikan:**

Gunakan batasan langsung di prompt:

```text
Task tiny. Minimal diff. Ubah 1 file ini saja. Jawaban ringkas.
```

### 6) Konvensi proyek tidak diikuti

**Gejala:** style/pattern yang dipakai tidak sesuai kode existing.

**Langkah perbaikan:**

1. Sebutkan konvensi secara eksplisit di prompt (mis. "gunakan useApi", "jangan ubah naming").
2. Referensikan file contoh yang harus diikuti.
3. Minta agent melakukan revisi dengan scope sempit pada file terkait.
