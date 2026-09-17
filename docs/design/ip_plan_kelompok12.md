# Skema IP — Kelompok 12
### TEK1314 Keamanan Siber · Pertemuan 4 · Design Phase

**Kelas:** [TEK A1]
**Anggota:**
- [Dinda Kahrnita] — [0404241003] (Lead)
- [Radithya Hafizh Kaila Hermawan] — [J0404241160] (Blue Team)
- [Ahmad Jawawi] — [J0404241089] (Blue Team)
- [Daffa' Raihandi] — [J0404241038] (Red Team)

---

## Skenario

Kelompok 12 mengambil skenario **Web Server**: Target Server akan menjalankan aplikasi web yang sengaja dibuat rentan terhadap **SQL Injection** dan **XSS**. Instalasi belum dilakukan minggu ini — tahap ini fokus pada perancangan topologi dan alamat IP saja.

## Alokasi Subnet

Kelompok 12 menggunakan blok `192.168.12.0/24` (subnet mask `255.255.255.0`), unik dan tidak tumpang tindih dengan kelompok lain. Host address tersedia dari `192.168.12.1` sampai `192.168.12.254`, broadcast di `192.168.12.255`.

## Daftar Perangkat

| Perangkat | IP | OS | Catatan |
| --- | --- | --- | --- |
| Router/Gateway | 192.168.12.1 | Router-PT | Menghubungkan seluruh node dalam segmen |
| Target Server | 192.168.12.5 | Ubuntu Server CLI | Korban — akan menjalankan web app rentan |
| Attacker Node | 192.168.12.100 | Kali Linux | Dipakai Red Team untuk pengujian |
| Monitoring Node | 192.168.12.200 | Security Onion | Memantau seluruh trafik segmen |

Ubuntu Server CLI dipilih untuk Target Server karena paling ringan dibanding DVWA ISO atau Metasploitable, mengingat keterbatasan RAM laptop tim. Kemungkinan DVWA akan dipasang di atas Ubuntu ini nanti supaya celah SQLi/XSS lebih gampang didemonstrasikan — masih didiskusikan, belum keputusan final.

## Alur Trafik yang Direncanakan

Secara garis besar ada dua arah trafik yang relevan untuk skenario ini:

1. **Attacker → Target Server** — trafik pengujian/serangan, lewat HTTP (80) dan HTTPS (443), menyasar form input di web app.
2. **Target Server & Attacker → Monitoring Node** — semua trafik di atas ikut terekam Security Onion, karena node ini diposisikan untuk mengawasi seluruh segmen `192.168.12.0/24`, bukan cuma satu jalur.

## Riset Port (Red Team)

| Port | Layanan | Celah yang diincar |
| --- | --- | --- |
| 80 | HTTP | SQL Injection, XSS |
| 443 | HTTPS | Sama seperti port 80 + risiko SSL/TLS misconfigured |
| 22 | SSH | Brute-force, kalau skenario butuh akses admin tambahan |
| 3306 | MySQL | Password default/lemah, kalau DB jadi backend web app |



