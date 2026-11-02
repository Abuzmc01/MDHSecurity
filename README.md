# MDHSecurity 1.0.0

Plugin keamanan untuk network MineIDHost: login, premium auto-login, anti-bot, anti-grief, log blok + rollback, dan anticheat dasar.

| File | Pasang di | Versi |
|---|---|---|
| MDHSecurity-Velocity.jar | Proxy Velocity | 3.4+ (MC 1.21.11 ke atas) |
| MDHSecurity-BungeeCord.jar | Proxy BungeeCord / Waterfall | build terbaru (MC 1.21.11 ke atas) |
| MDHSecurity-Paper.jar | Backend Paper | 1.20.1 sampai terbaru |
| MDHSecurity-Purpur.jar | Backend Purpur | 1.20.1 sampai terbaru |

Catatan: isi kode Paper dan Purpur sama (Purpur memakai API Paper). Dipisah supaya mudah dibedakan di panel.

Butuh: **Java 17+** dan **MySQL / MariaDB**. Semua server (proxy + backend) memakai database yang sama.
Driver MySQL diunduh otomatis ke `plugins/MDHSecurity/libs/` saat pertama jalan kalau server belum punya.

---

## Cara pasang (network dengan proxy)

1. Buat database, mis. `minecraft`, dan satu user MySQL.
2. **Proxy**: taruh jar proxy di `plugins/`, start sekali, lalu isi bagian `database` di `plugins/mdhsecurity/config.yml` (Velocity) atau `plugins/MDHSecurity/config.yml` (Bungee). Restart.
3. **Wajib di proxy:** `online-mode = false` (Velocity: `velocity.toml`, Bungee: `config.yml`). Plugin yang memutuskan per pemain siapa yang dicek ke Mojang.
4. **Backend** (setiap server Paper/Purpur): taruh jar, start sekali, isi `database` dan `server-name` (beda tiap server) di `plugins/MDHSecurity/config.yml`. Biarkan `auth.proxy-mode: true`. Restart.
5. **Wajib supaya backend tidak bisa dimasuki langsung tanpa proxy:**
   - Velocity: `player-info-forwarding-mode = "modern"`, lalu di backend `config/paper-global.yml` → `proxies.velocity.enabled: true` + `secret` yang sama.
   - BungeeCord: `ip_forward: true`, backend `spigot.yml` → `bungeecord: true`, dan **firewall** port backend supaya hanya proxy yang bisa konek.
   Tanpa ini, orang bisa masuk ke backend langsung dan melewati pengecekan premium.

## Server tanpa proxy

Pakai jar Paper/Purpur saja dan set `auth.proxy-mode: false`.
- `online-mode=true` di server.properties: semua pemain premium, login otomatis.
- `online-mode=false`: semua pemain Java pakai /register dan /login. Mode premium otomatis hanya tersedia lewat proxy.

---

## Cara kerja login

- **Java premium**: proxy mengecek nama ke Mojang. Kalau nama itu akun asli, proxy memaksa verifikasi Mojang. Pemain asli langsung masuk tanpa password. Client crack yang memakai nama premium tidak bisa masuk.
- **Java non-premium**: masuk ke server tujuan dalam keadaan "beku" (tidak bisa jalan, chat, pakai perintah, buka chest, dipukul) sampai `/register` atau `/login`.
- **Bedrock (Geyser + Floodgate)**: otomatis login (nama berawalan `.` atau UUID Floodgate).
- Akun cracked yang sudah terdaftar tetap pakai password walaupun namanya kebetulan ada di Mojang. Kalau pemiliknya memang punya akun asli, dia bisa ketik `/premium`.
- Masuk lagi dari IP yang sama dalam `remember-ip-minutes` tidak perlu login. Pindah server di dalam network juga tidak perlu login ulang.
- Password disimpan sebagai PBKDF2-SHA256. Hash AuthMe (`$SHA$`) bisa dibaca, jadi data AuthMe lama bisa dipindah ke tabel `mdhsec_accounts`.
- Salah password `max-attempts` kali: kick + IP diblokir `block-minutes` menit.
- Password tidak ikut tercatat di console (`/login`, `/register`, `/changepassword` disembunyikan dari log).

**Anti-bot di proxy:** batas koneksi per IP per menit, filter nama, dan mode serangan otomatis. Kalau terlalu banyak nama baru masuk dalam 10 detik, hanya pemain yang sudah pernah terdaftar yang boleh masuk selama beberapa menit.

## Anti-grief

- **Pemain baru**: item seperti TNT, lava, flint & steel, end crystal, dan sejenisnya baru bisa dipakai setelah main sekian menit (default 120).
- **Proteksi spawn**: blok, chest/container, item frame, armor stand, lukisan, ledakan, dan api di radius spawn.
- **Hewan peliharaan**: tidak bisa dilukai pemain selain pemiliknya.
- **Log**: hancur/taruh blok, ember, ledakan (TNT dicatat atas nama pemain yang menyalakan), blok terbakar, ambil/masukkan item di container.

## Anticheat dasar

Cek fly, speed, reach, killaura (sudut serangan), autoclicker (CPS), dan fastbreak. Default hanya memberi notifikasi ke staff dan melakukan setback atau membatalkan pukulan. `command-vl` default 0, artinya tidak ada kick otomatis.

Pemain Bedrock, creative/spectator, elytra, kendaraan, air/tangga, efek potion tertentu, dan sesaat setelah teleport atau knockback tidak dicek.

Ini bukan pengganti anticheat besar seperti Vulcan atau Grim. Kalau sudah memakai salah satunya, matikan `anticheat.enabled`.

---

## Perintah

| Perintah | Fungsi | Permission |
|---|---|---|
| /register `<pw> <pw>` | Daftar | semua |
| /login `<pw>` | Login | semua |
| /changepassword `<lama> <baru>` | Ganti password | semua |
| /premium → /premium confirm | Pindah ke auto-login premium (UUID berubah) | semua |
| /mdhsec inspect | Klik blok untuk lihat riwayat | mdhsec.inspect |
| /mdhsec lookup `<pemain\|#tnt\|#fire\|*> [waktu] [radius]` | Cari riwayat | mdhsec.inspect |
| /mdhsec rollback `<pemain\|#tnt\|#fire\|*> <waktu> [radius]` | Kembalikan perubahan | mdhsec.rollback |
| /mdhsec undo | Batalkan rollback terakhir | mdhsec.rollback |
| /mdhsec alerts | Nyala/matikan notifikasi anticheat | mdhsec.alerts |
| /mdhsec info `<pemain>` | Data akun + VL | mdhsec.admin |
| /mdhsec unregister `<pemain>` | Hapus akun | mdhsec.admin |
| /mdhsec changepass `<pemain> <pw>` | Ganti password pemain | mdhsec.admin |
| /mdhsec premium `<pemain> on\|off` | Atur mode premium | mdhsec.admin |
| /mdhsec forcelogin `<pemain>` | Paksa login | mdhsec.admin |
| /mdhsec reload | Muat ulang config | mdhsec.admin |

Format waktu: `30m`, `2h`, `3d`, `1w` (bisa digabung, mis. `1d12h`). Target `*` wajib memakai radius.

Bypass: `mdhsec.bypass.newplayer`, `mdhsec.bypass.spawn` (default OP), `mdhsec.bypass.pets`, `mdhsec.bypass.anticheat` (default tidak ada, OP tetap dicek).

---

## Batasan yang perlu diketahui

- Rollback tidak mengembalikan isi chest yang ikut hancur bersama bloknya. Item yang diambil/dimasukkan lewat GUI tercatat dan bisa dikembalikan. Gerakan hopper tidak dicatat.
- Pintu dan tanaman 2 blok dipulihkan utuh. Kasur dan blok multi-bagian lain bisa hanya satu bagian.
- `/premium` mengubah UUID pemain (offline → premium). Data plugin lain yang berbasis UUID (inventory, claim, ekonomi) bisa terpisah.
- Folia tidak didukung (khusus Paper/Purpur).
