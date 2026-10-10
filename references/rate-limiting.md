# rate-limiting.md — Batasi dengan sopan, bukan sekadar blokir

> Relevan: setiap API publik atau endpoint yang bisa di-abuse (login, kirim OTP,
> upload, endpoint mahal). Rate limit melindungi resource DAN tagihan.

## Pilih pola

| Pola | Cocok untuk | Catatan |
|---|---|---|
| Fixed window | Sederhana, cukup untuk kebanyakan kasus | Bisa "bocor" 2x lipat di batas window |
| Sliding window | Akurasi lebih baik | Butuh penyimpanan state |
| Token bucket | Burst yang wajar + rata-rata terkendali | Paling fleksibel untuk API publik |

Untuk aplikasi kecil: fixed window per user/API key per menit sudah cukup.
Naikkan ke token bucket bila butuh burst (mis. 100/menit tapi boleh 20 sekaligus).

## Aturan implementasi

1. **Batasi per identitas, bukan per IP buta.** IP bisa shared (kantor, kampus).
   Urutan: API key/user ID dulu, IP sebagai fallback.
2. **Beda endpoint, beda limit.** Login/OTP ketat (mis. 5/menit); baca data
   longgar (mis. 100/menit); endpoint mahal paling ketat.
3. **Tanpa Redis? Bisa.** Simpan counter di database (tabel `rate_limits`:
   key, window, count) — cukup untuk skala kecil-menengah. Bersihkan window
   lama berkala.
4. **Kembalikan 429 yang informatif.** Header wajib:
   - `Retry-After: <detik>`
   - `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`
   - Body JSON dengan pesan jelas + kapan boleh coba lagi.
5. **Jangan hitung yang gagal auth ke kuota user** — tapi tetap batasi per IP
   agar tidak jadi celah brute force gratis.
6. **Allowlist internal.** Health check, cron internal, dan IP sendiri jangan
   kena limit.

## Saat merencanakan (Jalur A)

Tentukan matriks limit per endpoint sejak desain API. Dokumentasikan di
dokumen API agar konsumen tahu batasnya — bukan kejutan 429.

## Saat mengoptimasi (Jalur B)

Tambahkan limit ke endpoint yang terbukti di-abuse (lihat log: pola request
berulang dari satu identitas). Mulai longgar, ketatkan bertahap sambil pantau
false positive (user legit yang kena limit = bug).

## Anti-pattern (yang sering salah)

- Rate limit global satu angka untuk semua endpoint → endpoint murah ikut ketat
  atau endpoint mahal ikut longgar.
- 429 tanpa `Retry-After` → client retry membabi buta, makin parah.
- Limit per IP saja → satu kantor kena blokir gara-gara satu user nakal.
- Tidak ada limit di endpoint kirim OTP/email → tagihan jebol.

## Cara verifikasi

- Simulasi: kirim request melebihi limit → pastikan 429 + header lengkap.
- Pastikan user legit tidak kena limit pada pola pakai normal.
- Log menunjukkan penurunan abuse setelah limit dipasang.
