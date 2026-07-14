---
name: loop-prevention
description: Smart stuck detection for OpenCode agent pipelines — detects when agents are stuck without progress, without limiting productive iterations on large tasks
---

# Loop Prevention & Circuit Breaker Protocol

## Prinsip Utama

**Deteksi stuck, bukan batasi iterasi.** Task besar membutuhkan banyak iterasi. Hanya intervensi ketika tidak ada progress.

## Aturan Emas

| Situasi | Tindakan |
|---------|----------|
| Subagent return kode **berbeda** setiap kali | ✅ LANJUTKAN — progress |
| Subagent return kode **IDENTIK** 2x berturut-turut | ⛔ STUCK — eskalasi |
| Error **sama** muncul setelah "fix" | ⛔ STUCK — eskalasi |
| Bug fix 5+ percobaan, setiap kali beda | ✅ LANJUTKAN — progress |
| Pipeline UI bolak-balik 10x, selalu ada perbaikan | ✅ LANJUTKAN — progress |
| Analysis paralysis (5+ planning tanpa kode) | ⚠️ WARNING — tanya user |
| Scope creep (nambah fitur di luar scope) | ⚠️ WARNING — ingatkan scope |

## Stuck Detection Signals

1. **Same-result twice**: Output IDENTIK (file sama, kode sama) setelah diminta perubahan
2. **Same error**: Error baris/file/pesan yang sama persis setelah "diperbaiki"
3. **No files changed**: Subagent klaim selesai tapi 0 file berubah

## Escalation Template

```
## ⚠️ DETECTED: Stuck Pipeline

### Issue
{why it's stuck — same result, same error, no progress}

### Evidence
- Attempt 1: {result}
- Attempt 2: {result} (IDENTIK)

### Progress So Far
{what HAS been completed}

### Options
1. Provide new direction / specification
2. Approve current state
3. Try different approach or subagent
```
