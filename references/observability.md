# observability.md — Tahu saat ada yang rusak, sebelum user komplain

> Relevan: aplikasi yang berjalan di production. Optimasi tanpa observability =
> buta: tidak tahu apa yang lambat, apa yang error, kapan limit habis.

## 1. Logging yang berguna (bukan sekadar banyak)

1. **Struktur, bukan teks bebas.** Log sebagai JSON: timestamp, level, request
   id, user/tenant id, durasi, pesan. Teks bebas tidak bisa di-query.
2. **Request ID di setiap request.** Satu ID menelusuri alur dari edge sampai DB.
   Tanpa ini, debug production = menebak.
3. **Level yang disiplin.** ERROR = butuh tindakan; WARN = aneh tapi jalan;
   INFO = peristiwa penting; DEBUG = detail (matikan di production).
4. **Jangan log yang sensitif.** Password, token, API key, data pribadi —
   tidak pernah masuk log. Ini juga aturan security.
5. **Log yang mahal dimatikan di production.** Log per-baris di loop panas =
   boros I/O dan resource.

## 2. Error tracking

1. **Kumpulkan error otomatis** (Sentry gratis / self-host / sekadar agregasi
   log ERROR). Error yang tidak terkumpul = tidak pernah diperbaiki.
2. **Alert untuk yang penting saja.** ERROR rate naik tajam, endpoint kritis
   down. Alert untuk semua = alert fatigue = diabaikan semua.
3. **Sertakan konteks.** Stack trace + request id + input pemicu (tanpa data
   sensitif). "Error 500" tanpa konteks tidak bisa diperbaiki.

## 3. Metrik minimal

Pantau 4 angka ini (cukup untuk kebanyakan aplikasi kecil):
- **Error rate** — % request gagal (target < 1%).
- **Latency p95** — per endpoint penting.
- **Saturasi resource** — CPU/memori, koneksi DB, sisa kuota free tier.
- **Business critical** — 1-2 metrik inti aplikasi (mis. checkout berhasil).

## 4. Pantau limit free tier (khusus)

Free tier mati diam-diam saat limit habis. Catat dan pantau:
- Sisa kuota database (rows/storage), bandwidth, build minutes, function calls.
- Pasang peringatan di 70-80% — bukan saat sudah 100%.
- Punya rencana saat limit tercapai: optimasi, upgrade, atau degradasi graceful.

## Saat merencanakan (Jalur A)

Tentukan format log, request id, dan 4 metrik sejak awal. Jangan retrofit
observability ke aplikasi yang sudah besar — mahal.

## Saat mengoptimasi (Jalur B)

Nyalakan dulu: strukturkan log, pasang error tracking, ukur 4 metrik.
Observability adalah langkah 0 sebelum optimasi — tanpanya, "perbaikan"
tidak bisa dibuktikan.

## Anti-pattern (yang sering salah)

- Log teks bebas "masuk sini 1", "masuk sini 2" — tidak bisa dicari.
- Tidak ada request id — debug production dengan menebak.
- Alert ke semua channel untuk semua error — akhirnya dimute semua.
- Tidak tahu limit free tier sendiri sampai aplikasi mati.

## Cara verifikasi

- Setiap ERROR di production bisa ditelusuri: request id → log lengkap → penyebab.
- Dashboard (sesederhana apa pun) menampilkan 4 metrik, update berkala.
- Simulasi error → alert sampai ke yang bertanggung jawab dalam hitungan menit.
