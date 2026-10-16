# platforms.md — Optimasi aplikasi desktop & mobile (non-web)

> Dibaca bila target **bukan web**: aplikasi desktop (WPF / C# / C++ / Python),
> Android (Kotlin / Java), atau Flutter. Modul-modul umum tetap berlaku —
> prinsipnya portable; modul ini **menerjemahkan** prinsip itu ke metrik,
> pola, dan alat ukur tiap platform. Untuk prosedur kerja (Jalur A/B), tetap
> pakai `checklists.md`, ganti metrik web dengan metrik platform di bawah.

## Cara menentukan platform (deteksi cepat)

| Sinyal di repo | Platform |
|---|---|
| `*.sln`, `*.csproj`, `*.xaml` | .NET desktop (WPF / WinForms / WinUI) |
| `CMakeLists.txt`, `*.vcxproj`, `Makefile` + `*.cpp` | C++ desktop |
| `*.py` + `PyQt`/`PySide`/`tkinter`/`wx` import, `*.spec` (PyInstaller) | Python desktop |
| `build.gradle`/`build.gradle.kts`, `AndroidManifest.xml`, `*.kt`/`*.java` | Android (Kotlin/Java) |
| `pubspec.yaml`, `lib/main.dart` | Flutter |

Bila sinyal campur (mis. backend + aplikasi desktop), audit per komponen —
jangan paksa satu metrik untuk semua.

## 1. Desktop — WPF / C# / .NET

**Metrik:** waktu cold start (< 2–3 dtk), working set/RAM, UI thread tidak pernah
diblokir (> 50ms = terasa), ukuran installer.

1. **Jangan blokir UI thread.** Semua I/O pakai `async`/`await`; `ConfigureAwait(false)`
   di kode library; daftar ribuan item wajib virtualisasi (`VirtualizingStackPanel`,
   `IsVirtualizing="True"`) — jangan render 10.000 baris sekaligus.
2. **Startup cepat.** Lazy-load modul berat; publish dengan ReadyToRun / AOT bila
   memungkinkan; tampilkan splash screen hanya bila start > 2 dtk (bukan sebagai
   pengganti optimasi).
3. **Bereskan resource.** `IDisposable` selalu via `using`; unsubscribe event handler
   yang menahan referensi (penyebab leak klasik WPF); cache gambar pakai referensi
   lemah atau batas ukuran.
4. **WPF spesifik.** `Freeze()` untuk Freezable yang tidak berubah; hindari binding
   berat di dalam `DataTemplate`; `INotifyPropertyChanged` jangan tembak per properti
   dalam loop besar — batch.
5. **Single-file vs framework-dependent.** Single-file = startup lebih lambat
   (ekstraksi); pilih sesuai prioritas: distribusi mudah vs start cepat.

**Ukur dengan:** Visual Studio Diagnostic Tools, PerfView (gratis), dotTrace/dotMemory.

## 2. Desktop — C++

**Metrik:** cold start, RSS (resident memory), waktu frame (bila ada rendering),
ukuran binary.

1. **Hindari alokasi di hot path.** Pakai pool/arena/allocator khusus; `std::move`
   untuk transfer kepemilikan; `reserve()` sebelum loop push ke vector.
2. **Cache locality.** Struktur data datar (SoA) untuk loop besar; hindari pointer
   chasing di path panas.
3. **Kompiler bekerja untukmu.** Nyalakan optimasi rilis (`-O2`/`/O2`), LTO, dan
   PGO bila ada workload representatif — sering memberi 10–30% gratis.
4. **Exception & RTTI seperlunya.** Jangan pakai exception untuk alur kontrol di
   hot path; matikan RTTI bila tak dipakai (`-fno-rtti`).
5. **Startup.** Tunda inisialisasi berat (lazy); hindari static initializer yang
   mahal; link secukupnya (jangan bawa seluruh dunia via header).

**Ukur dengan:** `perf` (Linux), Tracy Profiler, Intel VTune; sanitizer
(ASan/TSan) untuk bug yang menyamar jadi "lemot".

## 3. Desktop — Python (PyQt / PySide / Tkinter / wx)

**Metrik:** waktu start (interpreter + import), RAM, UI tidak freeze.

1. **Import malas.** Pindahkan import berat (numpy, pandas, torch) ke dalam fungsi
   yang memakainya atau lazy loader — import di top-level menunda start.
2. **Jangan blokir event loop.** Kerja berat → `QThread`/worker + sinyal, bukan di
   slot yang dipanggil UI. UI freeze = aplikasi "not responding".
3. **Packaging.** PyInstaller `one-dir` start jauh lebih cepat dari `one-file`
   (one-file mengekstrak tiap start); Nuitka bila butuh lebih kencang.
4. **Hot path.** Hindari loop Python murni untuk data besar — vectorize (numpy),
   atau pindahkan ke C/Rust via binding. `cProfile` dulu sebelum menebak.
5. **Gambar & resource.** Kompres gambar; jangan bundle file raksasa yang jarang dipakai.

**Ukur dengan:** `py-spy` (profiling tanpa ubah kode), `tracemalloc`, `cProfile`.

## 4. Android — Kotlin / Java

**Metrik:** cold start, ANR rate (< 0,47%), crash-free (> 99,5%), ukuran AAB/APK,
baterai, jank (frame > 16ms).

1. **Main thread suci.** Tidak ada I/O, query DB, atau komputasi berat di main
   thread — itu sumber ANR. Pakai coroutines (`Dispatchers.IO`), `WorkManager`
   untuk kerja background (dengan constraint: charging/wifi bila cocok).
2. **Startup.** Tunda inisialisasi library (`App Startup` + lazy); Baseline Profile
   untuk AOT; splash via `SplashScreen` API, bukan activity palsu.
3. **Ukuran.** R8/ProGuard (minify + shrink + obfuscate); Android App Bundle;
   WebP untuk gambar; hapus resource tak terpakai (`shrinkResources`).
4. **Gambar & list.** Coil/Glide dengan ukuran sesuai layar; `RecyclerView` +
   `DiffUtil`; jangan inflate layout berat di `onBindViewHolder`.
5. **Database lokal.** Room + index untuk query filter/sort; jangan query di main
   thread; WAL mode untuk konkurensi baca-tulis.
6. **Baterai.** Batasi wake lock & alarm; batch network; lokasi hanya saat perlu
   dengan akurasi secukupnya.

**Ukur dengan:** Android Studio Profiler, Macrobenchmark, Firebase Performance,
Android Vitals di Play Console.

## 5. Flutter

**Metrik:** ukuran APK/IPA/AAB, waktu first frame, jank (terutama shader
compilation), startup, RAM.

1. **Ukuran.** Build release selalu App Bundle / `--split-per-abi`; tree-shake
   icon & font (`--tree-shake-icons`); `--obfuscate` + `--split-debug-info`.
2. **Jank.** `const` constructor di mana mungkin; `ListView.builder` (jangan
   `Column` + ribuan anak); `RepaintBoundary` untuk area animasi; hindari kerja
   di `build()` — pindahkan ke `initState`/cache; pertimbangkan Impeller.
3. **Gambar.** `cached_network_image`; resize ke ukuran tampil
   (`cacheWidth`/`cacheHeight`); WebP.
4. **State.** Jangan `setState` di root untuk perubahan kecil; pilih state
   management seperlunya (jangan over-engineering).
5. **Platform channel.** Batch panggilan native; jangan chatty (tiap call ada
   overhead crossing).

**Ukur dengan:** Flutter DevTools, `flutter build apk --analyze-size`,
Firebase Performance.

## 6. Lintas platform native (berlaku untuk semua di atas)

1. **Crash reporting sejak hari pertama.** Sentry / Firebase Crashlytics /
   Bugsnag. Aplikasi native yang crash diam-diam = user hilang tanpa kabar.
2. **Secret & data sensitif.** Jangan hardcode API key di binary (mudah di-extract);
   pakai Keystore (Android) / Keychain (Apple) / DPAPI (Windows) / libsecret (Linux).
   Obfuscation ≠ enkripsi — anggap secret di client sebagai publik.
3. **Auto-update.** Desktop: Squirrel / WinSparkle / MSIX / Sparkle (macOS).
   Tanpa update mudah, perbaikan performa tidak sampai ke user.
4. **Izin seminimal mungkin.** Minta izin (lokasi, file, notifikasi) hanya saat
   dibutuhkan dan jelaskan alasannya — penolakan izin = fitur "rusak" di mata user.
5. **Uji di perangkat nyata.** Emulator ≠ HP kentang user; uji di perangkat
   low-end untuk ANR, jank, dan baterai.

## Anti-pattern lintas platform

- Logika berat di UI thread / main thread ("cuma sebentar kok").
- Gambar full-res untuk thumbnail; tidak ada caching gambar sama sekali.
- Binary/APK membengkak karena semua dependensi dibawa "buat jaga-jaga".
- Secret/API key hardcode di kode atau config yang ikut ter-bundle.
- Crash tanpa laporan — "di HP saya aman-aman saja".
- Optimasi tebakan tanpa profiler ("kayaknya di sini yang lambat").

## Cara verifikasi per platform

- **Desktop .NET:** PerfView / VS Diagnostics — start < 3 dtk, tidak ada blokir UI thread.
- **C++:** `perf` / Tracy — hot path tanpa alokasi, PGO memberi gain terukur.
- **Python:** `py-spy` — tidak ada fungsi UI yang makan > 50ms; start membaik setelah lazy import.
- **Android:** Vitals — ANR < 0,47%, crash-free > 99,5%; Macrobenchmark untuk start & jank.
- **Flutter:** `--analyze-size` turun; DevTools timeline tanpa jank merah.
- Semua: crash reporting aktif dan ada crash pertama yang tertangkap (bukti jalannya),
  bukan "nol crash" karena tidak terpasang.
