# serverless.md: Ekonomi platform serverless (Vercel dkk.)

> Dibaca bila aplikasi jalan di platform serverless/edge (Vercel, Netlify,
> Cloudflare Workers, dsb.). Di sini "performa" = "tagihan": unit billing
> platform adalah metrik yang wajib dioptimasi, sama seperti Web Vitals.
> Ditulis dari kasus nyata: ISR Writes 50K/200K dalam 30 hari di blog Next.js
> yang kontennya cuma berubah beberapa kali sehari.

## 1. Kenali unit billing platformmu (wajib pertama)

Tiap platform menagih hal berbeda. Tulis di ADR sejak hari pertama:

| Platform | Unit yang membengkak diam-diam |
|---|---|
| Vercel | ISR writes/reads, Fluid Active CPU, Bandwidth (Origin/Data Transfer), Function size, invocations |
| Netlify | Function invocations + runtime, Bandwidth |
| Cloudflare Workers | Requests, CPU ms, KV reads/writes |

Prinsip: tiap kenyamanan ("biar fresh", "revalidate kecil saja") punya harga.
Harga itu harus tertulis sebelum fitur dibangun, bukan ditemukan di dashboard.

## 2. Strategi revalidasi: sumber ISR writes nomor satu

1. **Samakan interval dengan ritme konten.** Konten berubah 3x sehari → `revalidate = 86400`
   cukup. `revalidate = 3600` untuk konten harian = 24x writes yang tidak perlu.
2. **Sitemap/RSS/feed: harian atau on-demand, JANGAN per jam.** Ini pelaku klasik:
   file kecil yang diregenerate tiap jam padahal isinya berubah seminggu sekali.
3. **On-demand revalidation (`revalidatePath`/`revalidateTag`) hanya saat konten
   benar-benar berubah.** JANGAN di middleware atau route yang jalan per request,
   itu mesin cetak ISR writes.
4. **`generateStaticParams` untuk slug penting.** `dynamicParams = true` (default)
   + traffic bot/crawler = tiap URL unik yang disentuh = 1 ISR write. Blog yang
   dilahap AI crawler butuh ini.
5. **Diagnostik writes vs reads.** Normal: reads jauh lebih besar dari writes.
   **Writes > reads = ada yang salah** (revalidasi tanpa yang baca, atau bot
   memicu regenerasi terus-menerus).

## 3. Function diet: Fluid CPU, storage, cold start

1. **Kecilkan bundle function.** Import seperlunya di route handler; satu library
   berat yang diimport tapi jarang dipakai = storage + cold start tiap deploy.
2. **Jangan kerja berat di request.** Resize gambar, agregasi, kirim email →
   cron/queue, bukan di dalam request (Fluid CPU menagih per milidetik aktif).
3. **`force-dynamic` hanya untuk route yang memang harus dinamis.** Tiap route
   dinamis = function invocation tiap request, tanpa cache CDN.

## 4. Cache hit ratio CDN

- Target: CDN menyerap > 90% traffic halaman yang bisa statis.
- Kalau Origin Transfer tinggi padahal halamanmu statis → header cache salah
  (`s-maxage` tidak terpasang, atau `force-dynamic` di mana-mana).
- Ukur: bandingkan CDN Requests vs Origin Transfer di dashboard.

## 5. Alert kuota: observability biaya

- Catat baseline dashboard usage tiap minggu (angka, bukan "kayaknya naik").
- **Alert manual di 50% dan 75% kuota** bila platform tidak menyediakan alerting.
  Jangan tahu dari dashboard setelah jebol.
- Setiap fitur baru: estimasi dampak ke unit billing, tulis di PR/ADR.
  "Fitur ini menambah ~X ISR writes/hari" = bagian dari definisi selesai.

## Saat merencanakan (Jalur A)

Pilih strategi caching + revalidasi sebelum nulis kode: interval per tipe halaman,
apa yang on-demand, estimasi writes/hari. Masukkan ke budget di `performance.md`.

## Saat mengoptimasi (Jalur B)

1. Buka dashboard usage → identifikasi unit yang paling dekat ke limit.
2. Petakan ke kode: `grep revalidate` di semua route/page; cek cron yang memicu
   revalidasi; cek traffic bot di log.
3. Perbaiki interval → tunggu 7 hari → bandingkan angka sebelum vs sesudah.

## Anti-pattern (kasus nyata)

- `revalidate = 60` "biar fresh" di semua halaman, termasuk yang berubah mingguan.
- Sitemap/RSS regenerate per jam untuk konten harian (24x writes sia-sia per hari).
- `revalidatePath` di tiap publish untuk puluhan path yang tidak berubah.
- Tidak tahu angka dashboard sampai kuota jebol, lalu menyalahkan platform.
- Mengecilkan revalidate untuk "memperbaiki" konten basi, padahal yang benar
  adalah on-demand revalidation saat publish.

## Cara verifikasi

- Dashboard usage 7 hari sesudah vs sebelum: ISR writes turun, writes kembali
  di bawah reads.
- Tiap klaim menyebut angka: "ISR writes 1.667/hari → 210/hari", bukan "lebih hemat".
