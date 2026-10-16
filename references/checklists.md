# checklists.md — Prosedur audit: pre-build & existing

> Ini prosedur kerja agent. Jalankan sesuai jalur. Setiap item: cek →
> catat temuan (dengan angka/bukti) → perbaiki → verifikasi.

## A. Pre-build — Jalur A (aplikasi baru, sebelum/saat membangun)

Jalankan sekali di awal, dan ulangi tiap milestone besar.

### A1. Budget & fondasi (`performance.md`)
- [ ] Budget ditetapkan: LCP, INP, CLS, p95 API, bundle JS, ukuran halaman.
      Untuk desktop/mobile/Flutter: pakai metrik platform (`references/platforms.md`).
- [ ] Pola async diputuskan: apa yang jalan di background vs di request.
      Untuk native: apa yang boleh di main/UI thread vs wajib background.
- [ ] Timeout didefinisikan untuk semua panggilan keluar.

### A2. Database (`database.md`)
- [ ] Skema dirancang; index untuk FK + kolom filter/sort/join sudah ada sejak migrasi pertama.
- [ ] Strategi pooling/koneksi untuk target deploy (serverless?) diputuskan.
- [ ] Pola paginasi dipilih (cursor untuk data besar).

### A3. Caching (`caching.md`)
- [ ] Matriks data panas: data → lapisan cache → TTL → pemicu invalidasi. Ditulis, bukan di kepala.

### A4. Rate limiting (`rate-limiting.md`)
- [ ] Matriks limit per endpoint didokumentasikan di dokumen API.

### A5. UX (`ux.md`)
- [ ] Sistem desain mini: spacing, palet, 3 level tipografi, komponen states (loading/empty/error).

### A6. Security (`security.md`)
- [ ] 10 poin checklist security direncanakan (auth, validasi, headers, secrets, upload, dst.).

### A7. Observability (`observability.md`)
- [ ] Format log + request id + 4 metrik + pemantau limit free tier dirancang.

Hasil: dokumen desain + budget tercatat. Lanjut bangun.

---

## B. Audit existing — Jalur B (aplikasi sudah berjalan)

### B1. Ukur baseline (wajib pertama)
- [ ] Web Vitals: LCP, INP, CLS (Lighthouse, 3x run, catat median).
      Untuk desktop/mobile/Flutter: metrik platform (`references/platforms.md`) —
      cold start, ANR/crash-free, ukuran binary/APK, jank.
- [ ] Respons API p95 per endpoint penting.
- [ ] Query DB lambat: slow query log / pg_stat_statements, urut by total time.
- [ ] Bundle JS + ukuran halaman awal.
- [ ] Error rate dan pola error dominan.

### B2. Database (`database.md`)
- [ ] EXPLAIN untuk 5 query terlambat — index tepat guna?
- [ ] Cek N+1: jumlah query per request untuk endpoint utama.
- [ ] Transaksi: tidak ada I/O lambat di dalam transaksi.

### B3. Caching (`caching.md`)
- [ ] Data yang dihitung berulang teridentifikasi; hit rate potensial dihitung.
- [ ] Cache existing: TTL + invalidasi terdokumentasi dan terbukti jalan.

### B4. Frontend (`performance.md`)
- [ ] Gambar: WebP/AVIF? responsif? lazy? width/height terisi?
- [ ] Bundle: dependensi tak terpakai dibuang; splitting per rute.

### B5. Rate limiting + Security (`rate-limiting.md`, `security.md`)
- [ ] Endpoint sensitif (login/OTP/upload/mahal) punya limit + 429 informatif.
- [ ] 10 poin security checklist lolos.

### B6. UX (`ux.md`)
- [ ] 5 layar tersering: CLS OK? states (loading/empty/error) ada? feedback aksi < 100ms?

### B7. Observability (`observability.md`)
- [ ] Log terstruktur + request id jalan; error tracking aktif; 4 metrik terpantau.

### B8. Perbaiki berurutan (prioritas 80/20 di SKILL.md)
Untuk tiap temuan: satu perbaikan → ukur → catat angka → lanjut.
Jangan perbaiki dua hal sekaligus (tidak tahu mana yang berpengaruh).

### B9. Penjaga regresi (`debugging.md`)
- [ ] Angka sebelum/sesudah tercatat untuk tiap perbaikan.
- [ ] Budget performa jadi gate (CI bila ada; cek manual tiap rilis bila tidak).

Hasil: laporan audit (temuan + angka + perbaikan + verifikasi). Selesai bila
target tercapai atau prioritas habis — bukan bila "sudah dioptimasi semua".
