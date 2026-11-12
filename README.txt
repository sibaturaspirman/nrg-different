CAPTURE A DIFFERENT SIDE OF YOU – Photo kiosk TV portrait (4K)
================================================================

STYLE: Taste Experience – background biru glitch (assets/bg-a.webp & bg-b.webp), tombol gambar glitch (btn.webp),
judul dengan efek glitch merah/cyan, tagline di kiri atas, panel bersudut potong.
Versi style "Art for the Curious" (tombol pill cyan, background key visual client) disimpan terpisah di
../photo-different-kiosk_art-for-the-curious (jalankan di port 5701).

ISI FOLDER
- index.html : aplikasinya
- assets/    : background, tombol, font IQOS (style sama dengan Taste Experiences)
- lib/qrcode.js : pembuat QR (jalan tanpa internet)

FLOW
(Copy Nov 2026: Start/Choose "Capture moments that go beyond" + NEXT, Brief "3 Shots. 3 Expressions." + START CAPTURE,
 Capture "Show your curious side!", Result "Your beyond moment is ready".)
1. Start      – "Capture moments that go beyond" + READY
2. Consent    – "Before you continue..." + OK, LET'S GO!   (kamera mulai menyala setelah ini)
3. BARU: Choose your frame – 3 pilihan efek, semua dengan background assets/fx-bg.webp + glow biru di
   pinggiran orang: FACET (segitiga cahaya cyan-biru yang memudar + grid tipis), GRID (grid garis tipis +
   garis cahaya grid-light.png + sedikit segitiga), TRACK (kotak wajah tipis bercahaya + shape kaca
   track-glass.webp + garis cahaya). Cek tampilan filter di http://localhost:5700/lab/ (Filter Lab).
   Tiap kartu menampilkan preview live strip dengan efek itu.
4. Brief      – "3 Shots. 3 Chances."
5. Capture    – foto 3x di area foto (countdown 3-2-1), efek frame langsung terlihat di preview.
6. Result     – strip 3 foto berderet (header "Designed for the curious" + footer "Create beyond.") + QR.
               Ini layar terakhir; FINISH kembali ke awal. Di layar tampil 2 strip bertumpuk (dicetak 2 keping);
               QR upload = foto 9:16 (1080x1920); print = lembar 4x6 (2 strip berdampingan, 2400x3600).

CARA PAKAI
- Buka index.html lewat Chrome. Kamera butuh izin; buka dari http://localhost (atau https),
  karena Chrome memblokir kamera kalau dibuka langsung dari file:// di sebagian perangkat.
  Contoh: npx http-server photo-different-kiosk -p 5700  lalu buka http://localhost:5700
  Mode kiosk: chrome --kiosk --use-fake-ui-for-media-stream "http://localhost:5700"
- Desain 1080x1920, otomatis diskalakan x2 di TV 4K portrait (2160x3840). Strip hasil dibuat 1200x3600 px.
- Kalau kamera tidak terdeteksi, aplikasi masuk DEMO MODE (wajah ilustrasi) supaya flow tetap bisa dicoba.

PANEL ADMIN (tap pojok kiri atas / tombol 1 / Ctrl+M)
- Jarak foto: tombol 2 / Ctrl+L / tombol di panel admin
- Saat result: lembar 4x6 otomatis dibuka dialog print sistem.
- Kamera, mirror, glow biru di sekitar orang (default OFF), reset otomatis, orientasi layar, layar penuh.
- Upload endpoint + Authorization: default API IQOS (photo-ai-iims.zirolu.id).
  POST multipart name/phone/file (foto 9:16); response {"file":"https://..."} → URL itu jadi QR.
- Link QR cadangan: dipakai kalau upload gagal / offline.
- Download print 4x6 terakhir: file untuk dicetak (2 strip berdampingan, 2400x3600).
- Download foto 9:16 (QR) terakhir: file yang di-download pengunjung lewat QR (1080x1920, frame
  assets/download-frame.webp + 3 foto). File inilah yang dikirim ke API upload.
- Jarak foto (juga Ctrl/Alt+L): atur posisi foto di strip. Tab PREVIEW = strip di layar, PRINT KIRI / PRINT KANAN =
  strip kiri / kanan di kertas 4x6. GESER X = geser seluruh strip (foto + frame) ke kiri/kanan; LEFT, SIZE, TOP, GAP =
  posisi kiri, lebar, posisi atas foto pertama, dan jarak antar foto. Tersimpan otomatis; RESET DEFAULT = kembali
  ke posisi template untuk tab yang aktif.

CATATAN PENTING
- QR "Scan to download" berisi URL publik dari API upload (field "file").
  Kalau offline / upload gagal, dipakai Link QR cadangan (atau ID sesi).

DETEKSI ORANG & WAJAH (MediaPipe, Google)
- Saat dibuka, aplikasi memuat model segmentasi orang + deteksi wajah dari internet
  (cdn.jsdelivr.net dan storage.googleapis.com, ~1 detik). Ini yang membuat background terpisah rapi dari orang
  dan kotak/titik wajah di efek TRACK tepat di mata, hidung, dan mulut.
- Kalau kiosk offline, efek tetap jalan dengan cara lama (perkiraan dari warna kulit), hasilnya kurang rapi.
- Auto-framing: kamera otomatis zoom & mengikuti wajah (maks. 2.2x) supaya wajah selalu besar di frame,
  walau pengunjung berdiri agak jauh. Butuh model deteksi wajah di atas (online).
- Remove background (default ON, bisa dimatikan di panel admin > "Hapus background"): ruangan di belakang
  pengunjung diganti background filter (assets/fx-bg.webp) sebelum efek diterapkan, jadi hasil akhir fokus ke
  orang + efek. Ganti file itu untuk mengganti background semua efek.
