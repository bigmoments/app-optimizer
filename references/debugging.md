# debugging.md — Debug masalah performa & bug terkait optimasi

> Relevan: saat ada masalah performa atau bug yang muncul dari/terkait optimasi.
> Modul ini mandiri — tidak bergantung ke skill lain.

## Alur debug (selalu sama)

1. **Reproduksi dulu.** Buat masalahnya terjadi secara konsisten. Tidak bisa
   direproduksi = tidak bisa diverifikasi perbaikannya.
2. **Ukur, jangan tebak.** Profiling / EXPLAIN / log — tunjukkan dengan data
   di mana waktu habis. "Kayaknya di sini" bukan diagnosis.
3. **Isolasi.** Perkecil ruang lingkup: matikan cache → masih lambat? Matikan
   satu query → membaik? Binary search penyebabnya.
4. **Perbaiki satu hal.** Satu perubahan per iterasi, ukur dampaknya.
5. **Verifikasi + regresi.** Masalah hilang DAN tidak ada yang rusak
   (jalankan test / cek metrik terkait).

## Kasus khas terkait optimasi

| Gejala | Kemungkinan penyebab | Cek |
|---|---|---|
| Lambat kadang-kadang | Cache miss massal / cold start | Hit rate, pola waktu kejadian |
| Lambat setelah deploy | Index hilang, query plan berubah, bundle membengkak | Diff migrasi, EXPLAIN, bundle size |
| Data tidak update / basi | Invalidasi cache gagal | TTL, pemicu invalidasi, key version |
| Makin lambat seiring waktu | Memory leak, tabel tanpa vacuum, log menumpuk | Profil memori, ukuran tabel, rotasi log |
| Timeout acak | Tanpa timeout di panggilan keluar / transaksi lama | Audit semua I/O: ada timeout? |
| Boros resource tiba-tiba | Retry membabi buta, N+1 baru dari fitur baru | Log: pola request berulang? |

## Memory leak (pola umum)

Gejala: memori naik terus, tidak turun setelah beban reda. Cara isolasi:
heap snapshot berkala → objek apa yang bertambah? Penyebab klasik: listener
yang tidak dihapus, cache tanpa batas/eviction, koneksi yang tidak ditutup,
interval/timer yang tidak dibersihkan.

## Regresi performa

Setiap perbaikan performa wajib punya "penjaga":
- Catat angka sebelum/sesudah (p95, query time).
- Bila ada CI: budget performa sebagai gate (gagal bila jebol).
- Bila tidak ada CI: cek manual tiap rilis untuk endpoint kritis.

## Saat merencanakan (Jalur A)

Siapkan sejak awal: cara reproduksi lokal yang mirip production (seed data
cukup besar), akses ke profiling, dan definisi "lambat" (angka, bukan rasa).

## Saat mengoptimasi (Jalur B)

Jangan perbaiki lebih dari satu hal sekaligus. Dokumentasikan tiap temuan:
gejala → data → penyebab → perbaikan → angka sesudah. Ini jadi pengetahuan
tim, bukan sekadar "sudah dibetulkan".

## Anti-pattern (yang sering salah)

- Restart sebagai "perbaikan" tanpa tahu penyebab → akan kembali.
- Menambah resource (scale up) untuk menutupi bug → mahal dan sementara.
- Menebak penyebab lalu rewrite modul → 2 minggu, masalah tetap ada.
- Mematikan cache/observability agar "tidak error" → buta total.

## Cara verifikasi

- Masalah terreproduksi → terukur → terisolasi → terperbaiki → terverifikasi.
- Angka sebelum/sesudah tercatat.
- Penjaga regresi dipasang agar tidak terulang.
