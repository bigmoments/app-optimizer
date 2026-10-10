# security.md — OWASP esensial: aman itu bagian dari optimasi

> Relevan: selalu sebelum rilis. Aplikasi cepat tapi bocor = gagal total.
> Security yang baik juga menghemat resource (mencegah abuse = mencegah boros).

## Checklist wajib (tidak bisa ditawar)

1. **Validasi input di server.** Jangan percaya client. Tipe, panjang, format,
   dan batas nilai divalidasi di backend — selalu, tanpa kecuali.
2. **Parameterized query / ORM yang benar.** Jangan pernah gabung string ke
   query SQL. Ini anti SQL injection — tidak ada alasan melanggarnya.
3. **Output encoding anti-XSS.** Escape konten user saat dirender; Content
   Security Policy (CSP) sebagai pertahanan lapis kedua.
4. **Auth yang benar.**
   - Password: hash dengan bcrypt/argon2, jangan pernah MD5/SHA1 polos.
   - Session/token: expiry wajar, rotation, logout benar-benar membatalkan.
   - Jangan bikin sistem auth sendiri bila ada library teruji.
5. **CSRF protection** untuk cookie-based auth (token synchronizer / SameSite).
6. **Secrets tidak bocor.** API key, password, token JANGAN di repo, log, error
   message, atau URL. Pakai environment variables / secret manager.
7. **Security headers.** Minimal: `Strict-Transport-Security`,
   `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY` (atau
   `frame-ancestors`), `Referrer-Policy`, CSP yang masuk akal.
8. **Upload file aman.** Validasi tipe (magic bytes, bukan ekstensi), batasi
   ukuran, simpan di luar web root / object storage, jangan eksekusi.
9. **Error message aman.** Ke user: pesan umum. Detail (stack trace, query)
   hanya ke log internal.
10. **Dependency audit.** Jalankan `npm audit` / setaranya berkala; update
    dependensi dengan CVE kritis segera.

## Saat merencanakan (Jalur A)

Pilih strategi auth & session di awal (jangan ganti di tengah jalan).
Tentukan di mana secret disimpan dan siapa yang bisa akses. Security headers
dipasang sejak environment pertama.

## Saat mengoptimasi (Jalur B)

Audit 10 poin di atas satu per satu terhadap aplikasi berjalan — mulai dari
yang paling fatal (SQLi, secret bocor, auth). Perbaiki tanpa mengubah perilaku
user yang sah.

## Anti-pattern (yang sering salah)

- "Nanti saja security-nya, yang penting jalan dulu" → utang yang paling mahal.
- Validasi hanya di frontend (bypass dengan curl 5 detik).
- API key di-commit ke repo publik.
- Error 500 menampilkan stack trace + query SQL ke user.
- CORS `*` untuk API ber-auth.

## Cara verifikasi

- 10 poin checklist di atas lolos semua — tanpa kecuali sebelum rilis.
- `npm audit` (atau setara): nol CVE kritis.
- Uji manual: coba input `'<script>alert(1)</script>` dan `' OR '1'='1` di
  semua form — harus ditolak/dinetralkan.
- Header check: buka DevTools → pastikan security headers terkirim.
