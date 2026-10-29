# AxVision: Hawk — Changelog

Tanggal: 2026-10-29
Cakupan: `AxVisionD.sh` (daemon), `AxVisionF.sh` (feature watcher / SCR Engine), dan catatan perbaikan untuk `AxVisionC.sh` (CLI).

> Status pengujian: semua perubahan diuji dengan perintah Android palsu (stub) di lingkungan Linux biasa,
> plus `sh -n` (dash). **Belum pernah dijalankan di perangkat nyata.**

---

## AxVisionD.sh — ditulis ulang total

### Arsitektur baru
- **Journal pemulihan.** Setiap tweak game mencatat nilai *asli* sebelum ditulis, lalu dipulihkan persis saat keluar game
  (bukan menulis "nilai kebalikan"). Catatan disimpan di `/data/local/tmp/.visiond_journal`; kalau daemon mati di tengah game,
  start berikutnya memulihkan sendiri.
- **Snapshot config.** Seluruh config dibaca 1x per tick (`settings list global` + 1 `awk`), bukan 25+ `settings get`.
- **State machine game/daily.**
  - Masuk game langsung.
  - Keluar game dengan debounce 2 poll (overlay/notifikasi sesaat tidak memicu restore).
  - Proses game mati = keluar langsung.
  - Pindah antar game ditangani.
- **Ganti profil saat jalan.** Perubahan profil lewat CLI/UI diterapkan di tick berikutnya (sebelumnya hanya notifikasi).
- **Capability probe.** Perintah opsional dicek dulu (`cmd <svc> help`, `service check`) — wifi low-latency, ufw (UNISOC), MIUI mqsas.
- **Shutdown bersih.** `SIGTERM`/`SIGINT` memulihkan journal, boost game, dan governor CPU awal.

### Thermal
- `thermal_level()` berhisteresis: naik di ≥42 °C (level 1) dan ≥47 °C (level 2); turun di <44 °C (level 2) dan <40 °C (level 1).
- Profil pilihan user hanya bisa **diturunkan** oleh thermal, tidak pernah dinaikkan.
- **Safety floor:** tweak yang menurunkan proteksi suhu OEM (temp protect, XOS bypass, HyperOS disable thermal,
  disable thermal zone) dibatalkan otomatis saat baterai ≥45 °C.
- Suhu & persen baterai dibaca langsung dari sysfs (fallback `dumpsys battery`), tanpa CLI.

### Bug yang diperbaiki (ada di versi lama)
| Masalah | Perbaikan |
|---|---|
| `cmd game mode <pkg> performance` — urutan argumen terbalik (AOSP: mode dulu, baru paket), selalu gagal diam-diam | `cmd game mode performance <pkg>` / `standard <pkg>` |
| DNS dipaksa ke AdGuard selamanya setelah game | DNS asli dipulihkan dari journal |
| Data Saver dimatikan paksa saat restore walau awalnya aktif | Status awal dibaca (`cmd netpolicy get restrict-background`) lalu dipulihkan |
| Frame rate, driver game, `power_mode` tidak pernah dikembalikan | Semuanya lewat journal |
| `am freeze` + `set-inactive true` langsung setelah game bisa memutus game online saat user sebentar ke app lain | Dihapus; hanya mengembalikan yang diubah |
| `appops reset <pkg>` menghapus izin app-op lain yang diatur user | Diganti `appops set <pkg> WAKE_LOCK default` |
| `debug.hwui.renderer vulkan` (nilai tidak valid, komentar bilang sudah diperbaiki tapi kodenya tidak) | `skiavk` |
| Profil Saver memakai frekuensi **max** untuk cluster big | Memakai min (konsisten dengan maksud profil) |
| Status thermal diparse dari angka pertama di baris `mStatus` (kena angka suhu, mis. `49`) → `Invalid status: 49` | Hanya angka setelah `mStatus=`, harus 0..6; tidak valid = tidak override sama sekali |
| `dumpsys display \| grep -m1` → `Failed to write while dumping service display: Broken pipe` tiap loop | Output ditangkap ke variabel dulu |
| Notifikasi tidak muncul saat dijalankan lewat KSU/Magisk (uid 0 tercatat sebagai pkg "root") | Di uid 0, notif dikirim sebagai uid 2000 (`su 2000`) |
| Tweak game tidak dibatalkan kalau daemon mati di tengah sesi | Journal + recovery saat start |

### Penambahan
- Heads-up notification dimatikan selama game (nilai asli dipulihkan).
- `cmd wifi force-low-latency-mode` selama game (hanya jika ROM mendukung).
- Governor CPU diset per cluster (policy), governor yang tidak tersedia di kernel dilewati; governor awal dipulihkan saat daemon berhenti.
- BBR hanya dipasang jika kernel mendukung (`tcp_available_congestion_control`).
- `apply_multitask` idempoten (level sama tidak ditulis ulang).
- Job malam (cache cleaner, JIT, dexopt): jalan di background, ditunda jika layar menyala, dihentikan otomatis saat game dibuka.
- Reaper pakai lock PID (tanpa stale lock); log per-tick dihapus, hanya event penting yang dicetak.
- Notifikasi memakai newline asli (sebelumnya `\n` literal).
- Versi engine dibaca dari CLI baru (`vision --daemon --version[-code]`) dengan fallback ke nama lama (`--version-d[-code]`).

### Perilaku yang sengaja dipertahankan (opt-in oleh user)
- `preferred_network_mode=9` pada Network Adjustment **mematikan 5G selama game**; nilai asli dipulihkan saat keluar.
- Toggle thermal-disable, temp-protect, XOS bypass, HyperOS disable thermal tetap ada (dibatasi safety floor di atas).

---

## AxVisionF.sh — watcher fitur (SCR Engine)

### Bug yang diperbaiki (versi lama)
- `gms_state=(vision --gms --get)` adalah sintaks array, bukan command substitution → nilai tidak pernah terbaca.
- `[ a != b && c = 1 ]` tidak valid di `[ ]` → blok `if` tidak pernah jalan.
- Logika state terbalik (`local=0` setelah apply, `local=1` setelah restore) → akan apply/restore berulang tiap 10 detik.
- Notifikasi hilang di KSU/Magisk (lihat catatan uid 2000 di atas).

### Ditulis ulang sebagai watcher umum
Memantau toggle lewat settings, menerapkan/mengembalikan saat toggle berubah, tanpa restart daemon.
Nilai asli dicatat (journal `.visionf_journal`) lalu dipulihkan persis. Keadaan aktif dibaca dari jejak nyata, bukan ingatan proses.

| Fitur | Toggle | Efek |
|---|---|---|
| `gms` | `vision --gms --set 1\|0` | Matikan komponen analytics/measurement/usagereporting GMS (`vision --gms-opt`) |
| `anim` | `vision --feature anim --set 1\|0` | Animation scale 0.5x |
| `longpress` | `vision --feature longpress --set 1\|0` | `long_press_timeout` 205 ms (default 400) |
| `errreport` | `vision --feature errreport --set 1\|0` | `send_action_app_error` = 0 |
| `netwatch` | `vision --feature netwatch --set 1\|0` | `network_watchlist_enabled` = 0 |
| `ime` | `vision --feature ime --set 1\|0` | Compile keyboard aktif ke `speed`; `--reset` saat dimatikan |

- Fitur yang masih aktif ditegakkan ulang tiap 6 jam (OEM/update bisa mengembalikan nilai).
- Snapshot `settings list global` gagal → tick dilewati, tidak ada perubahan.
- Key setting (`long_press_timeout`, `network_watchlist_enabled`, `send_action_app_error`, tiga animation scale,
  `default_input_method`) dan namespace-nya sudah dicek ke source AOSP. `cmd package compile --reset` **belum**
  terverifikasi di source (compile sudah pindah ke modul ART).

### Sengaja tidak dimasukkan
`bloatware`, `deviceidle-whitelist`, `sqlite`, `low-ram-profile`, `surfaceflinger-timing`, `runtime-logging`,
`telementary-debloat` — efek samping besar (notif telat, risiko data rusak) atau belum bisa diverifikasi.

---

## AxVisionC.sh (CLI) — yang perlu diperbaiki

1. **Line ending CRLF** (`C` dan `D` di zip) → konversi: `sed -i 's/\r$//' AxVisionC.sh AxVisionD.sh`.
   Kemungkinan penyebab `Perintah tidak dikenali` (argumen berakhiran `\r`).
2. **`TMPD` tidak didefinisikan** → `GMS_STATE` menjadi `/.visiond_gms_disabled` (read-only), `restore` tidak bisa jalan.
   Tambahkan sebelum `GMS_PKG=`: `TMPD="${TMPD:-/data/local/tmp}"`.
3. **Versi daemon tertukar:**
   `ENGINE_VERSION_DAEMON="1.12.0-UDX-S"` dan `ENGINE_VERSION_DAEMON_CODE="10120"` (sebelumnya kebalikannya).
4. **Perintah baru** agar watcher mengenali fitur umum:
   ```sh
   --feature)   # vision --feature <anim|longpress|errreport|netwatch|ime> --set 1|0 / --get
     manage_state "Feature ($2)" "vision_feat_$2" "$3" "$4" "0"
     ;;
   ```
5. Perintah yang dipakai daemon dan masih ada di CLI: `--preload`, `--oxecute`, `--first_setup_ins`,
   `--gms`, `--gms-opt`, `--scr-engine`. Nama lama `--version-d` / `--version-d-code` sudah diganti
   `--daemon --version` / `--daemon --version-code`.

---

## Catatan & batas

- Tidak ada pengukuran CPU/baterai nyata. Klaim "lebih hemat" berasal dari berkurangnya fork per tick, bukan hasil ukur.
- Asumsi format output yang belum terbukti di perangkat: `settings list global`, `dumpsys content` (key refresh_rate),
  `dumpsys package` (daftar komponen GMS).
- `AxVisionT.sh` pada zip = `QiunixT.sh` v1.1 (CLI tweak) tanpa perubahan fungsional (hanya teks `about`).
