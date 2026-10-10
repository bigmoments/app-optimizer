# caching.md — Cache berlapis, invalidasi yang benar

> Relevan: data yang dihitung berulang, atau dibaca jauh lebih sering daripada ditulis.
> Cache adalah optimasi termurah setelah index DB — dan sumber bug paling licik.

## Kapan caching layak

- Rasio baca:tulis tinggi (mis. > 10:1).
- Perhitungan mahal (agregasi, render, panggilan pihak ketiga).
- Data boleh sedikit basi (tentukan toleransinya eksplisit, mis. "basi 60 detik OK").

Jangan cache: data yang harus selalu real-time (saldo, stok kritis), atau yang
jarang dibaca ulang (cache tidak pernah hit = sia-sia).

## Lapisan cache (dari dekat ke jauh)

1. **In-memory aplikasi** — paling cepat, untuk data kecil yang sering dibaca.
   Cocok bila tanpa Redis. Batas: hilang saat restart, tidak berbagi antar instance.
2. **HTTP cache** — header `Cache-Control`, `ETag`, `stale-while-revalidate`.
   Gratis bila ada CDN di depan. Untuk konten semi-statis ini lapisan termurah.
3. **CDN** — aset statis + halaman yang bisa di-cache di edge.
4. **Hasil query / computed** — simpan hasil agregasi berat dengan TTL.

Pilih lapisan paling luar yang memenuhi kebutuhan — jangan menumpuk semua lapisan
untuk satu data tanpa alasan.

## Aturan invalidasi (bagian tersulit)

Setiap cache WAJIB punya jawaban untuk tiga pertanyaan ini, ditulis eksplisit:

1. **Kapan basi?** — TTL berapa? (Contoh: 60 detik, 5 menit.)
2. **Kapan dihapus paksa?** — event apa yang memicu invalidasi? (Contoh: data
   di-update → hapus cache key terkait.)
3. **Apa yang terjadi saat miss?** — thundering herd: 100 request miss bersamaan
   jangan semuanya menghitung ulang. Pakai single-flight / lock /
   stale-while-revalidate.

Strategi penamaan key: `entitas:id:versi` — naikkan versi saat skema berubah,
jangan andalkan "nanti juga expired".

## Saat merencanakan (Jalur A)

Rancang matriks: tiap data panas → lapisan cache → TTL → pemicu invalidasi.
Tulis di dokumen desain. Jangan "nanti saja" — invalidasi yang tidak dirancang
= bug basi yang muncul 3 bulan kemudian.

## Saat mengoptimasi (Jalur B)

Tambahkan cache hanya setelah ukur: buktikan dulu data tersebut dihitung
berulang (hitung hit rate potensial dari log). Mulai dari satu lapisan terluar
yang masuk akal, ukur dampaknya, baru tambah lapisan bila perlu.

## Anti-pattern (yang sering salah)

- Cache tanpa TTL ("biar cepat") → data basi selamanya.
- Cache tanpa strategi invalidasi → "kok datanya tidak berubah-ubah".
- Men-cache yang jarang dibaca → memori penuh, hit rate nol.
- Key cache tanpa versi → deploy skema baru baca cache format lama = crash.
- Thundering herd saat cache expired massal.

## Cara verifikasi

- Hit rate terukur (log hit/miss) — target umum > 80% untuk data panas.
- p95 endpoint turun dengan angka konkret.
- Uji invalidasi: ubah data → pastikan cache ter-refresh dalam batas TTL yang dijanjikan.
- Uji skenario basi: matikan sumber data — aplikasi tetap jalan dari cache basi (bila dirancang begitu).
