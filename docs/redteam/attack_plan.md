# Attack Plan — Fase 2 (Target Node Replika BiliCheck, Kelompok 12)

Status: draf rencana untuk checkpoint Minggu ke-9. Eksekusi dan bukti lengkap (Purple Team Loop) dikerjakan mulai Minggu ke-10, dinilai formal di Minggu ke-11.

Target seluruh skenario: **Target Node replika** (`192.168.12.5`, Ubuntu Server + Flask + MySQL). Bukan sistem IoT/Firebase asli Mitra.

Ketentuan yang dipenuhi: 3 skenario dari 3 kategori berbeda (Reconnaissance, Exploitation/Akses, Web/API), tanpa skenario Availability/DoS.

## Skenario 1 — Reconnaissance: Port & Service Scanning

| Item | Detail |
|---|---|
| Kategori | Reconnaissance |
| Teknik | TCP SYN scan seluruh port, service/version detection, OS fingerprinting |
| Tools | nmap |
| Target | `192.168.12.5` (seluruh port) |
| Dasar riset Minggu ke-4 | Port yang relevan: 22 (SSH), 5000 (dashboard Flask) |
| Hasil yang diharapkan | Daftar port terbuka & versi service, dipakai sebagai dasar Skenario 2 dan 3; hasil diperkirakan tinggal `5000/tcp` terbuka ke internal network, `22/tcp` filtered dari posisi Attacker Node (lihat `hardening_checklist.md` kontrol B3) |
| Relevansi dengan tech stack | Memetakan permukaan serang nyata dari replika (Ubuntu + Flask), bukan asumsi generik |
| Deteksi yang diharapkan Blue Team | Alert Security Onion kategori scan (mis. "ET SCAN" / custom rule SYN Scan, sudah pernah teruji — lihat `docs/design/ip_plan.md`) |

## Skenario 2 — Exploitation/Akses: Brute-force & Pengujian Kontrol Akses

| Item | Detail |
|---|---|
| Kategori | Exploitation / Akses |
| Teknik | (a) Percobaan kredensial berulang (brute-force) terhadap SSH; (b) pengujian apakah endpoint dashboard Flask dapat diakses tanpa autentikasi (missing-authentication check) |
| Tools | Hydra (SSH), pengujian manual/`curl` terbatas untuk endpoint dashboard |
| Target | (a) SSH `192.168.56.10:22`, diuji dari segmen host-only (jalur admin yang sah) — **bukan** dari Attacker Node Kali di `192.168.12.0/24`, karena Skenario 1 dan kontrol B3 sudah menunjukkan port 22 di `192.168.12.5` filtered/timeout dari jalur itu; (b) dashboard Flask `192.168.12.5:5000`, diuji dari Kali |
| Dasar riset Minggu ke-4 | Hasil Skenario 1: port 5000 terbuka ke internal network, port 22 hanya reachable dari host-only (B3) |
| Hasil yang diharapkan | (a) Dari host-only: mengukur efektivitas `MaxAuthTries 3` (A2) dan ketiadaan fail2ban (A4 — Belum), sekaligus mengonfirmasi `PermitRootLogin no` (A1) menolak akun root; (b) Dari Kali ke `192.168.12.5:22`: percobaan diharapkan **gagal/timeout** sejak awal — ini dicatat sebagai bukti kontrol B3 sudah efektif (baseline positif untuk Purple Team Loop Tahap 1), bukan kegagalan skenario; (c) Pada dashboard: mengecek apakah route/endpoint yang ada memang tidak memiliki mekanisme login sama sekali — bila benar demikian, ini menjadi temuan baru (gap kontrol A untuk aplikasi web, belum tercakup di `hardening_checklist.md`) yang akan ditambahkan sebagai item checklist sebelum Minggu ke-11, bukan dipaksakan sebagai "brute-force login" yang belum tentu ada |
| Relevansi dengan tech stack | Menguji kontrol A2/A3/A4 (password auth SSH masih aktif, fail2ban belum dipasang — lihat `hardening_checklist.md`) dari jalur admin yang sah, sekaligus memverifikasi B3 dari sisi Attacker Node, dan mengisi gap dokumentasi soal auth pada dashboard |
| Deteksi yang diharapkan Blue Team | Alert banyak percobaan login SSH gagal dalam waktu singkat (dari host-only); untuk percobaan dari Kali ke port 22, yang diharapkan justru TIDAK ada log autentikasi karena koneksi diblokir di layer firewall (UFW) sebelum mencapai sshd |

## Skenario 3 — Web/API: Pengujian Input Dashboard

| Item | Detail |
|---|---|
| Kategori | Web/API/Protokol IoT |
| Teknik | Pengujian validasi input pada form/endpoint dashboard (kelas SQL Injection/XSS) |
| Tools | Burp Suite atau OWASP ZAP |
| Target | Endpoint dashboard Flask BiliCheck (port 5000) |
| Dasar riset Minggu ke-4 | Skenario 1 memastikan port 5000 aktif dan merupakan aplikasi web |
| Hasil yang diharapkan | Temuan validasi input yang kurang (jika ada), dicatat sebagai dasar kontrol C4 (validasi input); mengingat user DB aplikasi (`biliapp@localhost`) sudah dibatasi `SELECT`-only (C3), dampak SQLi yang mengubah data diperkirakan terbatas — tetap diuji untuk kemungkinan SQLi tipe baca (data exposure) dan XSS |
| Relevansi dengan tech stack | Menguji kontrol kategori C yang masih berstatus Belum di `hardening_checklist.md` |
| Deteksi yang diharapkan Blue Team | Log aktivitas mencurigakan pada aplikasi (kontrol C5, masih Belum — jadi kemungkinan belum terdeteksi di level aplikasi; diandalkan deteksi jaringan dari Security Onion) |

## Catatan Etika & Batasan
- Seluruh pengujian hanya terhadap Target Node replika di VM lab (`192.168.12.5` / `192.168.56.10`), bukan sistem/endpoint produksi Mitra.
- Data yang dipakai saat pengujian adalah data sintetis.
- Skenario 2 bagian (a) dari host-only mensimulasikan skenario "kredensial admin bocor/insider", bukan serangan dari jaringan publik — dicatat eksplisit agar Blue Team tahu jalur mana yang diuji.
- Skenario 2 dan 3 dijalankan setelah persetujuan jadwal dengan Blue Team, supaya Minggu ke-10–11 bisa langsung masuk siklus Purple Team Loop (serang → deteksi → analisis → respons → verifikasi) sesuai `docs/purple-team-log.md`.
- Tidak ada skenario Availability/DoS pada rencana ini.
