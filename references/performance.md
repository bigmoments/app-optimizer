# performance.md — Fondasi: budget, backend ringan, frontend ringan

> Relevan: selalu. Baca modul ini dulu sebelum modul lain.

## 1. Tetapkan budget (wajib sebelum optimasi)

Tanpa angka target, optimasi tidak punya garis finis. Tetapkan di awal (Jalur A)
atau ukur sebagai baseline (Jalur B):

| Metrik | Target wajar |
|---|---|
| LCP (Largest Contentful Paint) | < 2,5 dtk |
| INP (Interaction to Next Paint) | < 200 ms |
| CLS (Cumulative Layout Shift) | < 0,1 |
| Respons API p95 | < 500 ms (endpoint baca), < 1 dtk (endpoint tulis) |
| Bundle JS awal | < 200 KB gzip |
| Total halaman awal | < 1 MB |

**Saat merencanakan:** tulis budget di dokumen/ADR. Setiap fitur baru tidak boleh
menjebol budget tanpa diskusi eksplisit.
**Saat mengoptimasi:** ukur dulu dengan Lighthouse / WebPageTest / APM sederhana.
Catat angkanya — ini pembanding semua perbaikan.

## 2. Backend ringan

1. **Jangan lakukan di request yang bisa dilakukan di luar request.** Kirim email,
   resize gambar, agregasi berat → antrean/background job, bukan di dalam
   request-response.
2. **Kembalikan hanya yang dibutuhkan.** Hindari `SELECT *`; paginasi wajib untuk
   list (cursor-based lebih baik dari offset untuk data besar); batasi depth
   relasi yang di-expand.
3. **Hindari kerja berulang per request.** Inisialisasi berat (koneksi, parsing
   config) dilakukan sekali saat startup, bukan per request.
4. **Pilih format ringan.** JSON secukupnya; untuk payload besar pertimbangkan
   kompresi (gzip/brotli di level server/CDN).
5. **Timeout di semua panggilan keluar.** HTTP client, query DB, layanan pihak
   ketiga — semua wajib punya timeout. Tanpa timeout, satu layanan lambat
   menenggelamkan seluruh aplikasi.
6. **Cold start (serverless):** jaga bundle fungsi kecil, inisialisasi malas
   (lazy) untuk dependensi berat, hindari top-level await yang memblokir.

## 3. Frontend ringan

1. **Gambar adalah pemboros terbesar.** Pakai WebP/AVIF, `srcset` responsif,
   `loading="lazy"` untuk gambar di bawah lipatan, tentukan `width`/`height`
   untuk cegah layout shift.
2. **Kurangi JS awal.** Code splitting per rute; `defer`/`async` untuk script
   non-kritis; tree-shaking; audit dependensi — hapus yang tidak dipakai.
   Aturan praktis: jangan tambah library untuk beberapa baris kode.
3. **Font non-blocking.** `font-display: swap`, subset font, maksimal 2 family.
4. **CSS efisien.** Hindari CSS yang tidak terpakai; jangan kirim framework CSS
   utuh bila hanya pakai 10% nya.
5. **Minimalkan round-trip.** Gabungkan request yang bisa digabung; prefetch
   hanya untuk navigasi yang sangat mungkin terjadi.

## Anti-pattern (yang sering salah)

- Optimasi tebakan: "kayaknya lambat di sini" tanpa profiling → buang waktu.
- Micro-optimasi duluan (ganti `for` jadi `map`) sementara query N+1 dibiarkan.
- Menambah caching/infra untuk menutupi kode yang boros, bukan memperbaiki kodenya.
- Bundle JS 1 MB+ karena "biar gampang" pakai 5 library UI sekaligus.
- Gambar PNG 4K untuk thumbnail 200px.

## Cara verifikasi

- Lighthouse / PageSpeed: LCP, INP, CLS membaik dan di bawah budget.
- Respons API: ukur p50/p95 sebelum vs sesudah (bukan sekali, tapi beberapa kali).
- Bundle: `du -sh` / bundle analyzer — angka turun, bukan perasaan.
- Setiap klaim perbaikan wajib menyebut angka: "p95 1200ms → 320ms", bukan "jadi lebih cepat".
