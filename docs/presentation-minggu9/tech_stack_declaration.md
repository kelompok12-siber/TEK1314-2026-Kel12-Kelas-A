# Deklarasi Tech Stack — Bilicheck (Kelompok 12)

## Konteks Proyek
Proyek **Bilicheck** adalah sistem IoT untuk pengukuran bilirubin. Sensor memakai dua modalitas — **fotodioda** dan **LDR** — yang menghitung konsentrasi bilirubin lewat rumus absorbansi cahaya (mirip hukum Beer-Lambert). Data sensor dikirim dari ESP32 sebagai JSON (nilai mentah + hasil kalkulasi `jaundice_percentage`, `concentration`) melalui protokol **MQTT** ke broker `test.mosquitto.org:1883`, pada topic `biliona/esp32/sensor-data` (data), `biliona/esp32/status` (status), dan `biliona/esp32/command` (kontrol).

Pada rancangan asli, Firebase digunakan sebagai layanan backend/cloud dan Flutter (Dart) sebagai aplikasi klien. Pada PBL ini, proyek direplikasi terbatas untuk menguji hardening server, kontrol akses, dan monitoring keamanan jaringan — sensor fisik dan integrasi Firebase lengkap belum direplikasi di VM lab.

## Tabel Pemetaan Tech Stack Asli → Target Node Replika

| Komponen | Teknologi pada Proyek IoT Asli | Direplikasi sebagai (Target Node) |
|---|---|---|
| Sistem Operasi backend | Firebase (layanan backend/cloud terkelola; OS infrastruktur tidak dikelola langsung oleh tim) | Ubuntu Server 24.04.5 LTS (kernel `6.8.0-147-generic`) |
| Database/penyimpanan data | Firebase — Realtime Database untuk data streaming dari ESP32, Cloud Firestore untuk data terstruktur aplikasi mobile | MySQL 8.0.46 pada Ubuntu Server |
| Protokol komunikasi sensor | MQTT, broker `test.mosquitto.org:1883`, topic `biliona/esp32/sensor-data`, `biliona/esp32/status`, `biliona/esp32/command` | Komunikasi sensor (MQTT) belum direplikasi di VM lab. Dashboard simulasi diakses via HTTP pada TCP port 5000 |
| Dashboard/monitoring | Aplikasi klien Flutter (Dart) | Dashboard web Flask + Security Onion untuk monitoring keamanan jaringan |

## Catatan Penting
- Firebase adalah layanan backend/cloud terkelola, bukan sistem operasi.
- Flutter adalah framework aplikasi; Dart adalah bahasa pemrogramannya — keduanya bukan komponen backend.
- MySQL pada replika adalah pengganti simulasi lokal, bukan integrasi Firebase sesungguhnya.
- Seluruh data yang dipakai (tabel `users`, `pasien_bayi`, `pengukuran`) adalah data sintetis, bukan data pasien nyata, dan tidak boleh dipakai untuk keputusan medis.
- Broker MQTT `test.mosquitto.org` yang dipakai di firmware adalah broker publik/testing, bukan broker privat produksi.
