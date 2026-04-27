# Frontend Developer Agent (Nuxt) - Setup Guide

Panduan ini menjelaskan cara menggunakan agent di repo ini, termasuk skill yang perlu diinstall agar agent berjalan optimal.

## Gambaran Singkat

Repo ini berisi konfigurasi OpenCode untuk agent frontend khusus Nuxt/Vue:

- Agent config: `.opencode/config.json`
- Agent prompt: `.opencode/agents/frontend-developer.md`
- Dokumentasi internal: `.opencode/agent-docs/frontend/nuxt/frontend/nuxt/`
- Folder pendukung tim: `commands/`, `rules/`, `hooks/`, `contexts/`

Agent didesain untuk:

- Nuxt 4 + Nuxt UI
- Pola Vue Composition API
- Workflow operasional tim (scope-safe, verification status, commit/PR policy)

## Prasyarat

- OpenCode CLI sudah terpasang
- Akses ke repository ini
- Node.js + npm/pnpm/yarn/bun sesuai kebutuhan proyek

## Skill yang Digunakan Agent

Repo ini sudah menyertakan folder `skills/` (hasil sinkron dari environment lokal), jadi developer lain tidak perlu mencari skill satu per satu.

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

### Skill yang tidak perlu untuk operasional agent ini

Untuk proyek Nuxt frontend ini, skill backend/lintas bahasa di bawah tidak diperlukan agar agent berjalan optimal:

- `springboot-*`, `java-*`, `jpa-patterns`
- `django-*`
- `golang-*`
- `python-*`
- `clickhouse-io`, `postgres-patterns`, `backend-patterns`

Catatan: skill tersebut boleh tetap disimpan jika tim memang butuh multi-stack, tapi tidak wajib untuk agent frontend ini.

## Lokasi Skill

OpenCode biasanya membaca skill dari:

- `~/.opencode/skills/`
- `~/.agents/skills/`

Verifikasi cepat:

```bash
ls ~/.opencode/skills
ls ~/.agents/skills
```

Pastikan semua skill wajib tersedia.

## Cara Install Skill dari Folder `skills/` Repo

Jika skill belum ada di mesin developer, copy dari repo ini ke direktori skill lokal:

```bash
mkdir -p ~/.opencode/skills
cp -R ./skills/coding-standards ~/.opencode/skills/
cp -R ./skills/frontend-patterns ~/.opencode/skills/
cp -R ./skills/frontend-design ~/.opencode/skills/
cp -R ./skills/web-design-guidelines ~/.opencode/skills/
cp -R ./skills/nuxt-ui ~/.opencode/skills/
cp -R ./skills/security-review ~/.opencode/skills/
cp -R ./skills/tdd-workflow ~/.opencode/skills/
```

Opsional:

```bash
cp -R ./skills/building-components ~/.opencode/skills/
cp -R ./skills/vercel-composition-patterns ~/.opencode/skills/
```

## MCP yang Digunakan Agent

Dari `.opencode/config.json`, agent memakai MCP berikut:

- `nuxt` (enabled)
- `nuxt-ui` (enabled)
- `playwright` (enabled)
- `figma` (disabled by default, opsional)

Jika ingin pakai Figma MCP, set env var:

```bash
export FIGMA_ACCESS_TOKEN="your-token"
```

## Cara Menggunakan Folder Pendukung Tim

Penting: folder `commands/`, `rules/`, `hooks/`, `contexts/` di repo ini adalah **template bootstrap** (salinan dari `~/.opencode`) untuk mempermudah onboarding developer baru.

Folder-folder ini **bukan** otomatis aktif hanya karena ada di repo. Developer perlu copy/sinkron ke environment lokal masing-masing.

### `commands/`

- Berisi command/prompt siap pakai (biasanya slash command internal tim).
- Pakai sebagai koleksi command tim untuk task berulang agar hasil konsisten.
- Jika menambah command baru, ikuti format file yang sudah ada.
- Instal ke lokal jika diperlukan:

```bash
mkdir -p ~/.opencode/commands
cp -R ./commands/* ~/.opencode/commands/
```

### `rules/`

- Berisi aturan operasional/coding yang harus dipatuhi agent dan developer.
- Jadikan ini sebagai source of truth saat review code.
- Jika ada konflik antara preferensi pribadi dan `rules/`, ikuti `rules/`.
- Instal ke lokal sesuai README di `rules/README.md` (copy per direktori, jangan di-flatten).

### `hooks/`

- Berisi automasi lifecycle (pre-task/post-task, validasi ringan, dsb).
- Opsional: aktifkan hanya jika environment lokal mendukung hook dependencies-nya.
- File `hooks/hooks.json` memanggil script via `CLAUDE_PLUGIN_ROOT`; jika path/plugin tidak tersedia, nonaktifkan hook terkait.

### `contexts/`

- Berisi konteks reusable proyek (domain terms, batasan bisnis, pattern lokal).
- Rujuk konteks ini saat membuat prompt supaya agent tidak keluar jalur.
- Update saat ada perubahan requirement produk/arsitektur.
- Instal ke lokal jika ingin reuse context lintas repo:

```bash
mkdir -p ~/.opencode/contexts
cp -R ./contexts/* ~/.opencode/contexts/
```

### Praktik Tim yang Disarankan

- Gunakan `commands/` untuk pekerjaan rutin.
- Gunakan `rules/` sebagai standar wajib saat implementasi dan review.
- Gunakan `contexts/` untuk prompt yang lebih presisi.
- Perlakukan `hooks/` sebagai konfigurasi bersama tim (perubahan perlu sinkronisasi).

Jika tim tidak ingin maintain template global ini di repo proyek, folder-folder tersebut boleh dihapus tanpa mempengaruhi agent inti (`.opencode/`).

## Cara Pakai

Gunakan agent melalui mention di OpenCode:

```text
@frontend-developer Tambahkan UButton "Simpan" di app/components/profile/ProfileHeader.vue.
Task tiny, minimal diff, jangan ubah file lain.
```

Atau untuk task normal:

```text
@frontend-developer Implementasikan filter status di halaman markets.
Gunakan pola useApi yang sudah ada dan laporkan verification status.
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

Detail SOP tim ada di: `.opencode/agent-docs/frontend/nuxt/frontend/nuxt/TEAM_OPERATING_GUIDE.md`

## Referensi Cepat

- Quick start: `.opencode/agent-docs/frontend/nuxt/frontend/nuxt/QUICK_START.md`
- Dokumentasi utama: `.opencode/agent-docs/frontend/nuxt/frontend/nuxt/README.md`
- Prompt agent: `.opencode/agents/frontend-developer.md`

## Troubleshooting Instalasi Skill

### 1) Skill tidak muncul saat dicek

**Gejala:** nama skill tidak terlihat saat `ls ~/.opencode/skills` atau `ls ~/.agents/skills`.

**Langkah perbaikan:**

1. Pastikan path home benar:

```bash
echo $HOME
```

2. Cek dua direktori skill:

```bash
ls ~/.opencode/skills
ls ~/.agents/skills
```

3. Jika belum ada, install/copy skill ke salah satu direktori tersebut.

### 2) Skill ada, tapi agent tidak memakainya

**Gejala:** agent tidak memuat skill yang seharusnya dipakai.

**Langkah perbaikan:**

1. Panggil skill secara eksplisit di prompt:

```text
@frontend-developer Load skill `nuxt-ui` lalu implementasikan form ini.
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
