# Mentor Curriculum 30 Days (Nuxt Ecosystem Track)

Roadmap 1 bulan ini dirancang untuk developer yang **sudah bisa coding web dasar**, lalu ingin menguatkan fundamental yang relevan langsung ke stack:

- **Node.js**
- **Vue.js 3**
- **Nuxt.js 4**
- **Nuxt UI**
- **Tailwind CSS**

## Program Format

- Durasi: **4 minggu**
- Hari aktif: **5 hari kerja per minggu (Senin-Jumat)**
- Total sesi: **20 hari**
- Waktu harian: **90-150 menit**
- Komposisi: **25% konsep + 75% praktik**

## Target Outcome

Setelah 30 hari, peserta mampu:

1. Memahami runtime context Node.js untuk Nuxt development
2. Membangun feature Nuxt berbasis Vue Composition API
3. Menggunakan Nuxt UI secara konsisten untuk sistem antarmuka
4. Styling cepat dan maintainable dengan Tailwind CSS
5. Mengimplementasikan flow production dasar: data fetching, validation, error handling, testing mindset

---

## Minggu 1 - Node.js + Nuxt Runtime Fundamentals

**Tujuan minggu:** memahami pondasi runtime agar implementasi Nuxt tidak sekadar "jalan", tapi benar konteks server/client.

### Hari 1 (Senin): Node.js Runtime untuk Frontend Engineer
- Materi: event loop mental model, process lifecycle, env vars, npm scripts
- Praktik: bedah `package.json` scripts (`dev`, `build`, `preview`)
- Output: catatan alur eksekusi project dari `npm run dev` sampai app running

### Hari 2 (Selasa): Nuxt Execution Context (Server vs Client)
- Materi: SSR context, hydration concept, kapan kode jalan di server/client
- Praktik: buat contoh aman untuk akses browser API (`localStorage` guard)
- Output: 1 halaman demo server/client-safe behavior

### Hari 3 (Rabu): Nuxt Project Structure Deep Dive
- Materi: `app/`, `pages/`, `layouts/`, `components/`, `composables/`, `server/api/`
- Praktik: susun mini structure feature (list + detail)
- Output: struktur folder feature yang rapi

### Hari 4 (Kamis): Nuxt Routing + Layout + Middleware Basic
- Materi: file-based routing, nested route, route middleware basics
- Praktik: tambah auth-like middleware sederhana (dummy check)
- Output: flow navigasi + guard route dasar

### Hari 5 (Jumat): Nuxt Data Flow Fundamentals
- Materi: `useFetch` vs `useAsyncData` vs `$fetch`, kapan pakai masing-masing
- Praktik: implement 2 halaman data dengan strategi fetching berbeda
- Output: perbandingan singkat plus keputusan teknik

---

## Minggu 2 - Vue 3 + TypeScript Applied Fundamentals

**Tujuan minggu:** menguasai pola Vue modern dan type safety yang dipakai harian di Nuxt app.

### Hari 6 (Senin): Reactivity Architecture
- Materi: `ref`, `reactive`, `computed`, `watch`, efek ke re-render
- Praktik: filter/sort list berbasis computed
- Output: komponen list reactive dengan derived state bersih

### Hari 7 (Selasa): Component Contracts
- Materi: props, emits, slots, one-way data flow
- Praktik: buat komponen card reusable + slot action
- Output: 2-3 komponen reusable dengan contract jelas

### Hari 8 (Rabu): Composable Design
- Materi: extract logic dari komponen ke composable
- Praktik: `useProducts` atau `useMarkets` composable
- Output: composable dengan loading/error state dan typed return

### Hari 9 (Kamis): TypeScript for Nuxt/Vue
- Materi: typing props, emits, API response, union + narrowing
- Praktik: tambah tipe pada flow data fetch + form state
- Output: modul feature typed end-to-end dasar

### Hari 10 (Jumat): Weekly Mini Build
- Materi: integrasi konsep minggu 2
- Praktik: build fitur "catalog/list + detail + action" kecil
- Output: mini feature dengan architecture Vue yang rapi

---

## Minggu 3 - Nuxt UI + Tailwind CSS Systematic Styling

**Tujuan minggu:** membangun UI yang konsisten, cepat, dan maintainable dengan Nuxt UI + Tailwind.

### Hari 11 (Senin): Nuxt UI Component-First Workflow
- Materi: mapping kebutuhan UI ke komponen Nuxt UI
- Praktik: refactor elemen custom ke `UCard`, `UButton`, `UInput`, `UBadge`
- Output: 1 halaman UI dengan adopsi Nuxt UI dominan

### Hari 12 (Selasa): Form Patterns with Nuxt UI
- Materi: `UForm`, validasi input, error presentation
- Praktik: form create/edit dengan state submit dan error
- Output: form flow lengkap (validasi + feedback)

### Hari 13 (Rabu): Tailwind Fundamentals for Nuxt
- Materi: utility-first workflow, spacing scale, responsive modifiers, state modifiers
- Praktik: styling halaman dengan utility classes konsisten
- Output: style guide mini (spacing + typography + color usage)

### Hari 14 (Kamis): Tailwind Maintainability Patterns
- Materi: class composition strategy, reusable class pattern, avoiding class chaos
- Praktik: rapikan class di beberapa komponen nyata
- Output: komponen lebih bersih dan mudah maintain

### Hari 15 (Jumat): UI Consistency Review Day
- Materi: visual consistency, accessibility quick audit
- Praktik: audit 1 flow halaman (list -> detail -> submit)
- Output: daftar perbaikan UI/UX prioritas tinggi

---

## Minggu 4 - Production Fundamentals (Nuxt App Delivery)

**Tujuan minggu:** menggabungkan semua skill menjadi mini app siap demo dengan standar engineering dasar.

### Hari 16 (Senin): API Integration Strategy
- Materi: project API pattern (`useApi`/fetch wrapper), error normalization
- Praktik: integrasi 2 endpoint (list + mutation)
- Output: feature data dengan success/error handling konsisten

### Hari 17 (Selasa): State Management Strategy
- Materi: local state vs shared state, kapan pakai Pinia/useState/composable
- Praktik: implement shared state untuk satu flow utama
- Output: state architecture ringkas + alasan pemilihan

### Hari 18 (Rabu): Performance + Accessibility Baseline
- Materi: lazy components, payload awareness, semantic/keyboard basics
- Praktik: optimasi 1 halaman berat + a11y cleanup
- Output: sebelum/sesudah (catatan improvement)

### Hari 19 (Kamis): Testing Mindset for Nuxt App
- Materi: apa yang dites di unit/integration/E2E
- Praktik: tulis test plan untuk flow kritikal
- Output: 8-12 test case prioritas (dengan level test)

### Hari 20 (Jumat): Final Demo & Review
- Materi: technical storytelling, trade-off explanation
- Praktik: demo mini app + jelaskan keputusan arsitektur
- Output: mini app + review checklist + backlog improvement

---

## Weekly Evaluation Rubric (Skor 1-5)

Nilai di akhir tiap minggu:

1. Pemahaman runtime dan framework
2. Kualitas struktur komponen/composable
3. Konsistensi Nuxt UI + Tailwind usage
4. Kualitas error handling dan state flow
5. Readiness ke praktik production

Wajib tulis:

- 2 kekuatan utama minggu ini
- 2 gap paling penting untuk ditutup
- 1 fokus wajib minggu depan

## Daily Session Template

Gunakan format ini tiap hari:

```markdown
Hari ke-[x]

1) Concept refresh (20-30 menit)
- [konsep yang ditargetkan]

2) Applied coding (60-100 menit)
- [task implementasi nyata]

3) Review & reflection (10-20 menit)
- What worked?
- What broke?
- What to improve tomorrow?
```

## Setelah 30 Hari (Next Track)

Pilih satu jalur lanjutan:

1. Bangun 2 mini apps Nuxt dengan scope berbeda (dashboard + form-heavy app)
2. Implement testing nyata (Vitest + Playwright) untuk 1 flow end-to-end
3. Dalami Nuxt server/api patterns + auth flow basics
4. Dalami performance profiling dan deployment readiness checklist

---

_Roadmap ini dipakai oleh mentor agent untuk pembelajaran fundamental yang lebih tepat sasaran ke ekosistem Nuxt, bukan kelas pemrograman web dari nol._
