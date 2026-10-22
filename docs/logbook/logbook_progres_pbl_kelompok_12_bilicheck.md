# Logbook Progres — PBL Kelompok 12 (Bilicheck)

**Mata Kuliah:** TEK1314 Keamanan Siber  
**Periode:** Minggu ke-4 s.d. Minggu ke-9

---

## Minggu ke-4 — Perancangan Arsitektur & Skema IP

- Blue Team menyusun topologi: Attacker (Kali Linux) – Target (Ubuntu Server) – Monitoring (Security Onion) – Admin host (Windows).
- Skema jaringan:
  - Internal Network: `192.168.12.0/24`
  - Host-only: `192.168.56.0/24` (SSH admin)
  - NAT: `10.0.2.0/24` (update paket)
- Red Team melakukan riset port potensial: `22` (SSH) dan `5000` (dashboard Flask/Bilicheck).
- OS Target disepakati menggunakan Ubuntu Server 24.04.5 LTS.
- **Output:** `docs/design/topology.png` dan `docs/design/ip_plan.md` diunggah ke GitHub oleh Lead.

## Minggu ke-5 — Pemilihan VM & Persiapan IP Plan

- Menentukan VM yang digunakan untuk topologi lab:
  - **Ubuntu Server** — Target Node
  - **Kali Linux** — Attacker Node
  - **Security Onion** — Monitoring Node
- Menyiapkan skema IP Plan untuk ketiga VM sebagai lanjutan dari desain Minggu ke-4.
- **Catatan yang perlu dilengkapi:** siapa yang menginstal setiap VM serta spesifikasi RAM dan storage yang digunakan.

## Minggu ke-6 — Integrasi Antar-VM

- Menghubungkan ketiga VM (Ubuntu, Kali, dan Security Onion) dalam satu jaringan sesuai IP Plan, dengan konfigurasi adapter Internal Network, Host-only, dan NAT.
- Melakukan verifikasi konektivitas antar-node melalui ping, akses dashboard, dan SSH.
- **Catatan yang perlu dilengkapi:** kendala integrasi yang dialami, misalnya konflik IP atau masalah adapter.

## Minggu ke-7 — Perbaikan Alert Security Onion

- Security Onion awalnya belum memunculkan alert dengan benar. Konfigurasi kemudian diperbaiki hingga alert berhasil muncul.
- Pengujian dilakukan menggunakan traffic ICMP (ping) dan pemindaian nmap dari Kali Linux. Aktivitas terdeteksi di Sguil sebagai event **“PBL - ICMP Traffic Detected”** dan **“PBL TEST - SYN Scan Detected”**.
- **Catatan yang perlu dilengkapi:** akar masalah alert tidak muncul pada awal pengujian dan langkah perbaikannya.

## Minggu ke-8 — UTS

- Ujian Tengah Semester (tertulis) dilaksanakan.
- Tidak ada aktivitas proyek PBL pada minggu ini.

## Minggu ke-9 — Checkpoint Fase 1 → Fase 2 (Case Study SOC Engagement)

Seluruh ketentuan dikerjakan sesuai Panduan PBL Pertemuan 9.

### 1. Deklarasi Tech Stack

- **OS backend:** Ubuntu Server 24.04.5 LTS.
- **Database:** MySQL 8.0.46 sebagai pengganti Firebase.
- **Dashboard:** Flask + Security Onion sebagai pengganti Flutter.
- **Protokol sensor:** MQTT belum direplikasi di VM lab.

### 2. Hardening Target Node

- Sebanyak **11 dari 17 kontrol minimum** telah diselesaikan:
  - Kategori A: 2/4 kontrol
  - Kategori B: 6/8 kontrol
  - Kategori C: 3/5 kontrol
- Hasil tersebut memenuhi standar minimum, yaitu setidaknya 8 kontrol yang tersebar di 3 kategori (Bagian VIII panduan).
- Bukti sebelum dan sesudah hardening disimpan di `docs/blueteam/evidence/`.

### 3. Live Demo

- Security Onion berhasil mendeteksi traffic ICMP ping dan nmap SYN scan dari Kali Linux.
- Alert yang muncul adalah **“PBL - ICMP Traffic Detected”** dan **“PBL TEST - SYN Scan Detected”**.

### 4. Attack Plan Fase 2

Tiga skenario disiapkan dari tiga kategori berbeda:

1. **Port/service scanning** menggunakan nmap.
2. **Brute-force SSH dan pemeriksaan autentikasi dashboard** menggunakan Hydra.
3. **Pengujian validasi input dashboard/SQLi-XSS** menggunakan Burp Suite atau OWASP ZAP.

Tidak ada skenario DoS, sesuai Bagian IX panduan.

### 5. Kendala dan Solusi

- Kondisi “sebelum hardening” sempat berubah. Kendala ini diatasi dengan mereproduksi rule UFW lama, kemudian menghapusnya kembali untuk keperluan bukti sebelum dan sesudah.
- Proses hardening sempat menutup akses yang sah. Solusinya adalah memisahkan jalur SSH melalui Host-only dan akses dashboard melalui Internal Network.

### 6. Pembagian Presentasi

- **Lead:** konteks proyek dan tech stack.
- **Blue Team:** arsitektur, progres hardening, dan live demo.
- **Red Team:** attack plan Fase 2.

### 7. Output yang Diunggah ke GitHub

Seluruh output diunggah sebelum sesi praktikum, meliputi:

- `docs/presentation-minggu9/tech_stack_declaration.md`
- `docs/presentation-minggu9/slides.pdf`
- `docs/blueteam/hardening_checklist.md`
- Bukti hardening di `docs/blueteam/evidence/`
- `docs/redteam/attack_plan.md`

---

## Rencana Minggu ke-10–11 (Fase 2)

- Menyelesaikan kontrol hardening yang tersisa:
  - A3 — menonaktifkan autentikasi SSH menggunakan password.
  - A4 — memasang dan mengonfigurasi fail2ban.
  - B7 — melakukan patch OS.
  - B8 — memeriksa akun default.
  - C4 — menerapkan validasi input.
  - C5 — menerapkan logging.
- Menjalankan **Purple Team Loop** untuk ketiga skenario dalam attack plan dengan tahapan: serang → deteksi → analisis → respons → verifikasi.
- Mencatat proses dan hasil di `docs/purple-team-log.md`.
- Menggunakan hasil kegiatan sebagai dasar penyusunan **Vulnerability Assessment Report** pada Minggu ke-11.
