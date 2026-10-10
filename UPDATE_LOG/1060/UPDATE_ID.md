# Change Log

Berlaku untuk Vision Daemon dan Vision CLI.

## Penambahan

- **Pengaturan lewat CLI untuk fitur yang sebelumnya tidak bisa diatur:** Auto Preload, OS Performance, Display Enhancement, Temp Protect, DND, HyperOS Disable Thermal, XOS Bypass High Temp, Disable Thermal, dan Smart Compact.
- **Pilihan driver grafis dan frame rate per game** sekarang bisa diatur dari CLI.
- **Reset Downscale:** downscale game bisa dikembalikan ke normal.
- **Menu bantuan** (`--help`) yang menampilkan semua perintah.
- **Smart Compact:** saat layar mati, memori sistem dirapikan otomatis.
- **Pemeriksaan suhu berkala selama game:** batas thermal ikut suhu asli HP, tidak lagi dikunci di satu nilai sepanjang sesi.
- **Mode jaringan asli HP disimpan** sebelum Network Adjustment aktif, lalu dikembalikan setelah game selesai.
- **Preload bisa dilakukan ulang setelah HP restart,** dan memori yang terkunci oleh game sebelumnya dilepas dulu.

## Fix

- **Thermal Protection:** sebelumnya tidak pernah aktif karena pembacaan suhu salah format. Sekarang HP otomatis turun ke Balance atau Saver saat suhu tinggi, lalu kembali ke profil game saat sudah dingin.
- **Batas thermal di profil Performance, Balance, dan Saver:** sebelumnya bisa menutupi pembatasan suhu dari sistem. Sekarang mengikuti suhu asli.
- **Disable Thermal:** sebelumnya tidak pernah menyala. Sekarang berfungsi dan bisa dikembalikan ke normal.
- **Pengaturan frekuensi CPU:** tiga profil sekarang memakai frekuensi cluster yang benar, sebelumnya salah membaca core yang bukan cluster besar.
- **Profil Balance:** grafis memakai renderer yang valid.
- **Game Driver:** sebelumnya memilih driver untuk satu game malah mengubah semua aplikasi. Sekarang hanya game yang dipilih. Pilihan ANGLE juga sekarang tersimpan dengan benar.
- **ADPF:** setelah game selesai, pengaturannya kembali ke bawaan sistem, bukan dipaksa ke nilai lain.
- **Mode Multitasking:** pengaturan RAM sekarang memakai perintah yang benar. Mode High mengembalikan pengaturan memori ke otomatis.
- **Network Adjustment:**
  - Data Saver sekarang benar-benar menyala dan mati.
  - Mode jaringan dikembalikan ke semula, bukan dipaksa ke mode tanpa 5G.
  - Pesan error di log berkurang.
- **Background Reaper:** tidak lagi macet permanen kalau prosesnya sempat terhenti di tengah jalan.
- **Re-compile jam 03:10:** hanya dijalankan sekali sehari, sebelumnya bisa terulang berkali-kali.
- **Preload:** sebelumnya hanya jalan sekali seumur hidup per game. Sekarang jalan lagi setelah HP restart.
- **Downscale:** nilai scale dan mode dicek dulu supaya tidak salah input, dan FPS yang kamu pilih benar-benar dipakai.
- **Battery Monitor:** tidak lagi error saat data baterai gagal dibaca, dan sumber daya dari dock dikenali.
- **Pengembalian setelah game (OS Performance dan XOS Bypass):** kembali ke kondisi normal dengan benar dan tidak lagi mengacaukan profil harian.
- **Deteksi layar nyala/mati:** tidak salah baca di HP yang punya lebih dari satu layar.
- **Error diam-diam saat game mulai:** fitur yang dipanggil tapi tidak ada tidak lagi menimbulkan kegagalan tersembunyi.

## Hapus

- **Pemadatan memori paksa pada game:** bentrok dengan Preload yang berjalan sebelumnya.
- **Pengaturan freezer lama:** tidak dikenal sistem Android, diganti dengan yang benar.
- **Penghapusan pengaturan driver grafis yang menimpa pilihan profil.**
- **Pengembalian pengaturan performa OS yang menimpa profil harian.**

## Catatan

- Belum diuji di perangkat.
- Fitur yang mematikan proteksi suhu (Temp Protect, HyperOS Disable Thermal, Disable Thermal) bisa membuat HP lebih panas. Aktifkan dengan hati-hati.
- Nilai boost loading di fitur Downscale sangat besar dan bisa membuat HP panas.
- Setelah game, DNS kembali ke AdGuard, bukan DNS awal.
- Satu fungsi di daemon (`cmd_game`) belum ada isinya.
