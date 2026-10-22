# Hardening Checklist — Target Node Bilicheck (Ubuntu Server, MySQL, Flask)

Status per **20 Oktober 2026 (Minggu ke-9)**. Standar minimum panduan: ≥8 kontrol, tersebar di 3 kategori (A/B/C), minimal 2 per kategori.

**Ringkasan: 11 kontrol Selesai (A: 2, B: 6, C: 3), 0 Sebagian, 6 Belum (rencana sebelum Minggu ke-11).**

Akses dashboard: `192.168.12.5:5000` (internal network, dari Kali). SSH admin: `192.168.56.10` (host-only).
Keterangan status: **Selesai** = sudah diterapkan dan ada bukti; **Sebagian** = diterapkan tetapi bukti belum lengkap; **Belum** = belum dikerjakan.

## A. Access & Authentication

| # | Kontrol | Status | Bukti (before → after) | Evidence file |
|---|---|---|---|---|
| A1 | Login root SSH dinonaktifkan | Selesai | `permitrootlogin without-password` → `no` | `sshd-config.txt`, `00-sebelum-hardening.txt`, `01-ssh-hardening.png`, `ssh-login-gagal.txt`, `07-ssh-root-login-gagal.png` |
| A2 | Percobaan autentikasi SSH dibatasi | Selesai | `maxauthtries 3`, `logingracetime 30` (via `sshd_config.d/00-hardening.conf`); koneksi diputus setelah percobaan gagal | `sshd-config.txt`, `01-ssh-hardening.png`, `ssh-login-gagal.txt`, `07-ssh-root-login-gagal.png` |
| A3 | Nonaktifkan password auth SSH (key-based) | Belum | `passwordauthentication` masih `yes` | `00-sebelum-hardening.txt` |
| A4 | fail2ban / rate-limit login | Belum | — | — |

Bukti percobaan login gagal setelah hardening: `ssh root@192.168.56.10` dari Windows host ditolak (`Permission denied`) dan koneksi diputus — lihat `ssh-login-gagal.txt`, `07-ssh-root-login-gagal.png`.

## B. Service & Attack Surface

| # | Kontrol | Status | Bukti (before → after) | Evidence file |
|---|---|---|---|---|
| B1 | Firewall UFW aktif, default deny incoming | Selesai | `deny (incoming)` | `ufw-status.txt`, `06-bukti-akhir-2-ss-ip-ufw.png` |
| B2 | Dashboard 5000/tcp hanya dari internal network | Selesai | Rule `5000/tcp` Anywhere (v4+v6) dan dari `192.168.56.0/24` dihapus → tersisa `192.168.12.5 5000/tcp on enp0s8` dari `192.168.12.0/24`; dari Kali `curl` → `HTTP 200` | `ufw-before-after.txt`, `ufw-status.txt`, `kali-verifikasi.txt`, `02-ufw-sebelum-sesudah-hapus-rule.png`, `nmap-before-after.txt`, `08-nmap-sebelum.png`, `09-nmap-sesudah.png` |
| B3 | SSH dibatasi ke adapter host-only | Selesai | Rule SSH dari `192.168.12.0/24` dan tanpa batas interface dihapus → tersisa `22/tcp on enp0s9` dari `192.168.56.0/24`; dari Kali port 22 `Connection timed out`; nmap: port 22 `open` → `filtered` | `ufw-before-after.txt`, `ufw-status.txt`, `kali-verifikasi.txt`, `nmap-before-after.txt`, `08-nmap-sebelum.png`, `09-nmap-sesudah.png` |
| B4 | MySQL tidak diekspos ke jaringan | Selesai | `ss -lntp` → hanya `127.0.0.1:3306` dan `127.0.0.1:33060` | `listening-ports.txt`, `06-bukti-akhir-2-ss-ip-ufw.png` |
| B5 | Flask hanya listen di adapter internal network | Selesai | `app.run(host="0.0.0.0")` → `host="192.168.12.5"`; `ss`: `0.0.0.0:5000` → `192.168.12.5:5000` | `flask-bind.txt`, `listening-ports.txt`, `03-flask-bind-192.168.12.5.png` |
| B6 | Service tidak berjalan sebagai root + sandbox systemd | Selesai | `User=kel-12`, `NoNewPrivileges=yes`, `ProtectSystem=full`, `PrivateTmp=yes`, `ProtectKernelTunables=yes`, `ProtectControlGroups=yes` | `service-status.txt`, `systemd-unit-file.txt`, `04-service-sandbox.png` |
| B7 | Update & patch OS/paket | Belum | — | — |
| B8 | Hapus akun/kredensial default | Belum | Perlu diperiksa (akun default DB/aplikasi) | — |

Bukti nmap dari Kali (`nmap -sS -p-`), sebelum vs sesudah: port terbuka **2 → 1** (`22/tcp`, `5000/tcp` → `5000/tcp`). Kondisi "sebelum" direproduksi dengan menambahkan kembali rule lama lalu menghapusnya — lihat `nmap-before-after.txt`.

## C. Data & Application

| # | Kontrol | Status | Bukti | Evidence file |
|---|---|---|---|---|
| C1 | Kredensial tidak hardcoded (env var) | Selesai | `app.py` baris 12: `os.environ["BILICHECK_DB_PASSWORD"]`; unit systemd memakai `EnvironmentFile=/etc/bilicheck.env` | `app-env-grep.txt`, `10-grep-hardcode.png`, `systemd-unit-file.txt`, `05-bukti-akhir-1-env-grants-unit.png` |
| C2 | Izin file environment dibatasi | Selesai | `/etc/bilicheck.env` mode `600`, pemilik `root:root` | `service-status.txt`, `05-bukti-akhir-1-env-grants-unit.png` |
| C3 | Least-privilege user database | Selesai | `biliapp@localhost` hanya `SELECT` pada `pasien_bayi`, `pengukuran`, `users` (tanpa INSERT/UPDATE/DELETE/DROP) | `mysql-grants.txt`, `05-bukti-akhir-1-env-grants-unit.png` |
| C4 | Validasi input pada endpoint API | Belum | — | — |
| C5 | Logging aktivitas mencurigakan | Belum | — | — |

Catatan C1: `grep -rniE "password|secret" app.py` hanya menemukan baris 12 (`os.environ["BILICHECK_DB_PASSWORD"]`), jadi tidak ada kredensial hardcode di `app.py`.

## Tambahan (tidak dihitung ke minimum)
- `Restart=on-failure`, `RestartSec=5` pada unit systemd — meningkatkan ketersediaan, bukan kontrol keamanan. Bukti: `service-status.txt`.

## Bukti Pendukung
- `00-sebelum-hardening.txt`, `00-sebelum-hardening.png` — kondisi sebelum: `5000/tcp on enp0s8` terbuka ke Anywhere (v4+v6), SSH diizinkan dari `192.168.12.0/24`, `permitrootlogin without-password`, Flask listen `0.0.0.0:5000`.
- `ip-addr.txt` — `enp0s3` NAT 10.0.2.15, `enp0s8` internal 192.168.12.5, `enp0s9` host-only 192.168.56.10.
- `kali-verifikasi.txt` — dari Kali: `curl` ke `192.168.12.5:5000` → `HTTP 200`; `nc` ke port 22 → `Connection timed out`.

## Rencana Sampai Minggu ke-11
1. A3: nonaktifkan `PasswordAuthentication` (setelah SSH key dipasang dan diuji dari host).
2. A4: pasang fail2ban untuk SSH.
3. (Selesai) Bukti login gagal dan nmap sebelum-sesudah sudah diambil.
4. B7: `sudo apt update && sudo apt upgrade`, catat versi sebelum/sesudah.
5. B8: periksa akun/kredensial default.
6. C4–C5: validasi input pada endpoint dan aktifkan log percobaan login gagal.
7. Pastikan aplikasi tetap berfungsi dengan user DB `SELECT`-only (uji fitur yang menulis data).

## Catatan
- Belum ada screenshot sesi SSH baru dari host (`192.168.56.1`) ke `192.168.56.10` setelah rule baru. Tambahkan jika ada.
- sshd masih listen di `0.0.0.0:22`; pembatasan akses dilakukan di UFW (`22/tcp on enp0s9`). Opsional: tambahkan `ListenAddress 192.168.56.10`.
