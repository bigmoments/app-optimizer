# database.md — Database: biang kerok performa nomor satu

> Relevan: setiap aplikasi yang punya database. Di aplikasi data-driven, 80%
> masalah performa tinggal di sini.

## Saat merencanakan (Jalur A)

1. **Desain skema + index sejak hari pertama.** Setiap foreign key dan kolom yang
   dipakai di `WHERE`/`JOIN`/`ORDER BY` wajib punya index. Index yang ditambah
   belakangan = migrasi berat di data besar.
2. **Pilih tipe data secukupnya.** Jangan `TEXT` untuk yang cukup `VARCHAR(50)`;
   jangan simpan JSON raksasa bila hanya 2 field yang dibaca.
3. **Rencanakan pooling sejak awal.** Serverless + database = wajib connection
   pooler (mis. PgBouncer) atau HTTP driver — koneksi langsung per fungsi =
   kehabisan koneksi.
4. **Pikirkan pola akses baca vs tulis.** Rasio baca-tulis menentukan strategi
   caching dan index (lihat `caching.md`).

## Saat mengoptimasi (Jalur B)

1. **Temukan query lambat dulu.** Nyalakan slow query log / `pg_stat_statements`;
   urutkan by total time, bukan hanya rata-rata. Perbaiki yang paling sering
   dipanggil + paling lambat.
2. **Baca EXPLAIN.** `EXPLAIN (ANALYZE, BUFFERS)` untuk query lambat. Cari:
   `Seq Scan` di tabel besar (tambah index), `Nested Loop` yang meledak
   (statistik basi → `ANALYZE`), sort di disk (tambah index atau kurangi data).
3. **Basmi N+1.** Satu query per baris = bom. Solusi: join/eager loading,
   `IN (...)` batching, atau dataloader. Gejala: jumlah query = jumlah baris.
4. **Paginasi yang benar.** Offset besar (`OFFSET 100000`) tetap scan 100rb
   baris. Untuk data besar pakai keyset/cursor pagination (`WHERE id > ?`).
5. **Transaksi sependek mungkin.** Jangan lakukan I/O lambat (HTTP call) di dalam
   transaksi DB — kunci ditahan, koneksi habis.
6. **Migrasi aman.** Tambah kolom nullable dulu, backfill bertahap, baru jadikan
   NOT NULL. Hindari lock tabel lama di production (`CREATE INDEX CONCURRENTLY`).

## Aturan index (ringkas)

- Index untuk: foreign key, kolom filter/sort/join yang sering.
- Jangan over-index: tiap index memperlambat tulis dan makan ruang.
- Ukur: `EXPLAIN` sebelum dan sesudah tambah index — pastikan dipakai planner.
- Index komposit: kolom equality dulu, baru range/sort — urutan menentukan.

## Anti-pattern (yang sering salah)

- `SELECT *` lalu filter di kode aplikasi.
- N+1 yang tidak disadari ("kan cuma 50 baris" — 50 baris × 200 user = 10rb query).
- Index di semua kolom "biar aman" → tulis jadi lambat, ruang membengkak.
- Migrasi `ALTER TABLE` berat di jam sibuk tanpa strategi.
- Koneksi DB dibuka per request tanpa pool di serverless.

## Cara verifikasi

- Slow query log kosong dari query > 500ms (atau di bawah ambang yang ditetapkan).
- `EXPLAIN` menunjukkan index scan, bukan seq scan, untuk query panas.
- Jumlah query per request turun (ukur via query counter/log).
- p95 endpoint berbasis DB membaik dengan angka konkret.
