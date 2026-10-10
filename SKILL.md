---
name: app-optimizer
description: >
  Optimasi aplikasi agar ringan, cepat, aman, dan nyaman dipakai — ramah free tier.
  Gunakan saat merencanakan aplikasi baru, mengaudit performa aplikasi yang sudah
  berjalan, sebelum deploy/rilis, saat menambah API endpoint, atau saat ada keluhan
  aplikasi "lemot", "berat", atau boros resource. Trigger chat: "optimasi aplikasi",
  "audit performa", "bikin lebih ringan", "aplikasi lemot", "cek kecepatan",
  "app-optimizer", "optimize my app", "performance audit", "make it faster".
license: MIT
version: 1.0.0
---

# app-optimizer

Bikin aplikasi ringan, cepat, aman, dan nyaman dipakai — tanpa bergantung ke layanan berbayar.

## Cara kerja skill ini

Skill ini dipakai dua arah:

1. **Dipicu dari chat** — user meminta langsung: "optimasi aplikasi ini", "audit performa", "bikin lebih ringan", "aplikasi lemot", "app-optimizer".
2. **Otomatis oleh agent** — saat kondisi berikut terpenuhi:
   - Merencanakan/membangun aplikasi baru → **Jalur A**
   - Mengaudit atau memperbaiki aplikasi yang sudah berjalan → **Jalur B**
   - Sebelum deploy/rilis → jalankan checklist pre-rilis di `references/checklists.md`
   - Menambah endpoint API baru → terapkan `references/rate-limiting.md` + `references/security.md`
   - Keluhan performa ("lemot", "timeout", "boros") → ukur baseline → `references/debugging.md`

## Pilih jalur

**Jalur A — Perencanaan (greenfield):** aplikasi baru. Bakar optimasi sejak desain, bukan ditempel belakangan.
Urutan: tetapkan budget → desain skema DB + index sejak hari pertama → rancang lapisan caching → desain rate limit → sistem UX → auth → observability.
Prinsip: *jangan bangun utang performa.*

**Jalur B — Optimasi existing (brownfield):** aplikasi sudah jalan. Audit dulu, ukur baseline, perbaiki bertahap tanpa merusak yang berjalan.
Prinsip: *ukur, jangan tebak; perbaiki, jangan rewrite.* Jangan ubah arsitektur kecuali data membuktikan perlu.

## Prinsip inti

1. **Ukur dulu.** Tanpa angka baseline, optimasi adalah tebak-tebakan.
2. **Pola umum, bukan stack-spesifik.** Contoh boleh menyebut stack tertentu, tapi aturannya harus portable.
3. **Termurah dulu.** Mulai dari yang murah dan berdampak besar (lihat prioritas 80/20).
4. **Jangan korbankan kebenaran.** Cache basi, data hilang, atau security yang dilonggarkan bukan "optimasi".
5. **Verifikasi setiap perubahan.** Tiap perbaikan harus bisa dibuktikan dengan angka, bukan perasaan.

## Urutan prioritas 80/20

Saat waktu atau tenaga terbatas, kerjakan berurutan — berhenti saat target tercapai:

1. **Ukur baseline** — Web Vitals (LCP, INP, CLS), waktu respons API p95, query DB lambat, ukuran bundle JS.
2. **Database** — index yang hilang + N+1. Biang kerok nomor satu aplikasi data-driven.
3. **Caching** — untuk data yang dihitung berulang dan jarang berubah.
4. **Frontend** — gambar (WebP, responsif) + ukuran bundle JS.
5. **Rate limiting + security headers** — murah, sekali pasang, melindungi dari abuse.
6. **UX states** — loading/empty/error yang jelas (mempengaruhi *persepsi* kecepatan).
7. **Observability** — logging + error tracking + pantau limit free tier.
8. **Micro-optimasi** — hanya bila data bilang perlu. Jangan dikerjakan duluan.

## Indeks modul

| Modul | Kapan dibaca |
|---|---|
| `references/performance.md` | Selalu — fondasi: budget, backend & frontend ringan |
| `references/database.md` | Ada database — indexing, EXPLAIN, N+1, pooling, migrasi |
| `references/caching.md` | Data dihitung berulang / dibaca jauh lebih sering daripada ditulis |
| `references/rate-limiting.md` | Ada API publik atau endpoint yang bisa di-abuse |
| `references/ux.md` | Ada UI — hierarki visual, states, mobile, aksesibilitas |
| `references/security.md` | Selalu sebelum rilis — OWASP esensial |
| `references/observability.md` | Aplikasi berjalan di production |
| `references/debugging.md` | Ada masalah performa atau bug terkait optimasi |
| `references/checklists.md` | Prosedur audit: pre-build (Jalur A) dan audit existing (Jalur B) |

Baca modul yang relevan saja — jangan semua sekaligus. Setiap modul punya bagian
"saat merencanakan" vs "saat mengoptimasi", daftar anti-pattern, dan cara verifikasi.

## Kompatibilitas

Markdown murni, SKILL.md format standar. Tanpa hooks, plugin, MCP, atau script.
Install: copy folder `app-optimizer/` ke direktori skill masing-masing harness —
mis. Claude Code (`~/.claude/skills/`), Cursor, Codex, Antigravity, Gemini CLI.
