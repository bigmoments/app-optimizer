# ux.md — UX profesional: ringan dirasa, nyaman dipakai

> Relevan: setiap aplikasi yang punya UI. Performa teknis tanpa UX yang baik
> tetap terasa lambat — persepsi kecepatan sama pentingnya dengan angka.

## 1. Persepsi kecepatan (lebih murah dari optimasi beneran)

1. **Skeleton > spinner.** Tampilkan kerangka konten saat loading, bukan spinner
   muter di tengah layar kosong. User merasa 30-50% lebih cepat.
2. **Optimistic UI untuk aksi ringan.** Like, toggle, simpan draf — tampilkan
   hasil langsung, sinkronkan di belakang. Rollback hanya bila gagal.
3. **Progressive rendering.** Tampilkan yang penting dulu (konten), tunda yang
   sekunder (komentar, rekomendasi).
4. **Feedback instan untuk setiap aksi.** Tombol berubah state saat diklik
   (< 100ms). Tanpa feedback = user klik berkali-kali = request ganda.

## 2. Hierarki visual & layout

1. **Satu aksi utama per layar.** Tentukan apa yang user harus lakukan di sini —
   tonjolkan itu, redupkan sisanya.
2. **Spacing konsisten.** Pakai skala spacing tetap (mis. kelipatan 4/8px),
   bukan angka acak per komponen.
3. **Tipografi sebagai hierarki.** Maksimal 2-3 ukuran font yang bermakna
   (judul, isi, keterangan) — bukan 7 ukuran berbeda.
4. **Jangan geser layout.** Tentukan ruang untuk gambar/iklan/konten dinamis
   di awal (lihat CLS di `performance.md`). Layout yang lompat-lompat =
   amatir.
5. **Dekat = berhubungan.** Kelompokkan yang berkaitan secara visual; pisahkan
   yang tidak. Jangan andalkan garis pembatas di mana-mana.

## 3. States yang wajib ada (sering dilupakan)

Setiap layar/data wajib punya desain untuk keempat state ini. Anggap masing-masing
sebagai "halaman" tersendiri seperti halaman 404 — bukan sekadar pesan tempel:

- **Loading** — skeleton, bukan layar kosong.
- **Empty** — ilustrasi/teks jelas + aksi ("Belum ada data — tambah pertama").
- **Error** — halaman error yang manusiawi:
  - Katakan apa yang terjadi dengan bahasa manusia ("Ada yang salah di sisi kami"),
    bukan kode ("Error 500").
  - Satu aksi pemulihan yang jelas ("Coba lagi" / "Kembali ke beranda").
  - Bedakan jenisnya: 404 (halaman tidak ada → tawarkan navigasi/pencarian),
    500/kesalahan server (minta maaf → coba lagi), error boundary (satu komponen
    rusak — jangan jatuhkan seluruh halaman).
  - Sertakan ID error untuk lapor ke support. User tidak bisa melapor dengan
    kalimat "tadi error".
- **Offline** — bedakan dua kasus, jangan disamaratakan:
  - *Aplikasi online-first* (butuh koneksi): state offline ala YouTube ("Anda sedang
    offline") — tampilkan jelas saat koneksi mati, nonaktifkan aksi yang butuh network
    dengan penjelasan ("Perlu koneksi untuk menyimpan"), sediakan "Coba lagi" dan/atau
    muat ulang otomatis saat koneksi kembali. Jangan biarkan user menatap spinner abadi.
  - *Aplikasi offline-first*: offline adalah mode operasi **normal**, bukan error.
    Aplikasi tetap jalan penuh; yang ditampilkan adalah **status sync** — "tersimpan
    lokal, menunggu sync" vs "sudah di cloud" — bukan larangan memakai aplikasi.
    Jangan nonaktifkan fitur yang bisa jalan lokal.
  - YAGNI: arsitektur offline-first (service worker/sync queue) hanya bila aplikasi
    memang harus bisa dipakai tanpa koneksi — jangan diresepkan ke semua aplikasi.

## 4. Mobile-first & aksesibilitas dasar

1. Desain untuk layar kecil dulu, baru lebarkan. Sentuh: target minimal 44px.
2. Kontras teks cukup (minimal 4.5:1 untuk teks isi).
3. Semua aksi bisa diakses keyboard; fokus terlihat jelas.
4. Jangan andalkan warna saja untuk informasi (tambah ikon/teks).
5. Hormati `prefers-reduced-motion` — matikan animasi berat bila diminta.

## Saat merencanakan (Jalur A)

Buat sistem desain mini dulu: skala spacing, palet warna, 3 level tipografi,
komponen states (loading/empty/error). Lebih murah daripada memperbaiki 50
layar yang tidak konsisten belakangan.

## Saat mengoptimasi (Jalur B)

Audit 5 layar paling sering dibuka: cek CLS, cek states yang hilang, cek
feedback aksi. Perbaiki yang paling sering dilihat user dulu.

## Anti-pattern (yang sering salah)

- Spinner fullscreen untuk loading 300ms.
- Tombol tanpa feedback → user klik 5x → 5 request.
- Empty state berupa layar putih kosong.
- Error berupa "Error 500" mentah ke user.
- 12 ukuran font, 9 warna biru berbeda, spacing acak.

## Cara verifikasi

- CLS < 0,1 di semua layar utama.
- Uji manual: matikan koneksi / throttle ke 3G — aplikasi tetap komunikatif.
- Setiap aksi punya feedback visual < 100ms.
- Minta 3 orang non-teknis pakai — catat di mana mereka bingung.
