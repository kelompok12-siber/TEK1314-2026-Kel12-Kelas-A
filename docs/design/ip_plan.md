# IP Plan — Simulasi Lab PBL Kelompok 12

Status: diperbarui untuk checkpoint Minggu ke-9 (20 Oktober 2026), mengikuti kondisi setelah hardening Fase 1.

## Ringkasan Jaringan

| Jaringan | Nama/Tipe Adapter | Subnet | Keterangan |
|---|---|---|---|
| Internal Network | `PBL-KEL12` (VirtualBox Internal Network) | `192.168.12.0/24` | Jaringan utama lab: Attacker, Target, Monitoring Node |
| Host-only | VirtualBox Host-only Adapter | `192.168.56.0/24` | Akses **SSH admin** dari Windows host ke Target Node (dashboard tidak lagi dibuka di jalur ini) |
| NAT | VirtualBox NAT | `10.0.2.0/24` (default VirtualBox) | Akses keluar/internet untuk update paket di Kali dan Ubuntu Server |

Bridged Adapter tidak digunakan dalam topologi ini.

Catatan NAT: Kali dan Ubuntu Server sama-sama memakai NAT VirtualBox dengan IP `10.0.2.15`. Ini tidak bentrok karena tiap VM mendapat jaringan NAT-nya sendiri; kedua VM tidak dapat saling mengakses lewat jalur NAT.

## Tabel Alokasi IP

| Node | Peran | Interface | IP Address | Jaringan | Fungsi |
|---|---|---|---|---|---|
| Kali Linux | Attacker Node | Adapter 1 | `10.0.2.15/24` | NAT | Akses internet (update paket/tools) |
| Kali Linux | Attacker Node | Adapter 2 | `192.168.12.100/24` | Internal Network `PBL-KEL12` | Menjalankan tools Red Team terhadap Target Node replika |
| Ubuntu Server | Target Node | `enp0s8` | `192.168.12.5/24` | Internal Network `PBL-KEL12` | Dashboard Flask BiliCheck (TCP 5000), diakses dari jaringan lab |
| Ubuntu Server | Target Node | `enp0s3` | `10.0.2.15/24` | NAT | Akses keluar untuk update paket OS (tidak ada service yang masuk) |
| Ubuntu Server | Target Node | `enp0s9` | `192.168.56.10/24` | Host-only | SSH admin dari Windows host (22/tcp) |
| Security Onion | Monitoring Node | `eth1` | `192.168.12.200/24` | Internal Network `PBL-KEL12` | Interface manajemen/analyst — dipakai aktif untuk Sguil, menerima alert |
| Security Onion | Monitoring Node | `eth0` | Tanpa IP (by design) | — | Flag `PROMISC` aktif — interface capture/sniffing lalu lintas lab |
| Windows Host | Client/admin | VirtualBox Host-only | `192.168.56.1/24` | Host-only | SSH ke Target Node |

## Aturan Akses (hasil hardening Fase 1)

| Layanan | Port | Interface / Bind | Sumber yang diizinkan (UFW) |
|---|---|---|---|
| Dashboard Flask | 5000/tcp | listen hanya di `192.168.12.5` | `192.168.12.0/24` lewat `enp0s8` |
| SSH | 22/tcp | listen di semua adapter, dibatasi firewall | `192.168.56.0/24` lewat `enp0s9` saja |
| MySQL | 3306 & 33060 | `127.0.0.1` saja | tidak dapat diakses dari jaringan |

UFW: aktif, default `deny (incoming)`, `allow (outgoing)`.
Konsekuensi: dashboard **tidak** dapat dibuka dari Windows host (`192.168.56.10:5000`), dan SSH **tidak** dapat dijangkau dari Kali (`Connection timed out`). Bukti: `docs/blueteam/evidence/kali-verifikasi.txt`, `ufw-status.txt`.

## Diagram Topologi
Lihat `docs/design/topology.png` untuk diagram visual (Attacker – Target – Monitoring Node, beserta jalur NAT dan Host-only).

## Konfigurasi Netplan (Ubuntu Server / Target Node)

```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: true
    enp0s8:
      dhcp4: no
      addresses:
        - 192.168.12.5/24
    enp0s9:
      dhcp4: no
      addresses:
        - 192.168.56.10/24
```

Perintah yang digunakan untuk menerapkan konfigurasi:
```bash
sudo netplan generate
sudo netplan apply
ip -br addr
```

## Verifikasi Konektivitas

```bash
# Dari Kali Linux ke Target Node (internal network)
ping -c 4 192.168.12.5
curl -I http://192.168.12.5:5000          # HTTP 200 (diizinkan)
nc -zv -w 3 192.168.12.5 22               # timeout (SSH diblokir dari internal network)

# Dari Windows host (host-only)
ssh kel-12@192.168.56.10                  # diizinkan

# Di Target Node
sudo ss -ltnp | grep ':5000'              # LISTEN 192.168.12.5:5000
curl -I http://192.168.12.5:5000
```

## Monitoring Security Onion
Monitoring sudah dibuktikan berfungsi melalui pengujian traffic nyata:
- Uji ICMP (ping) dari Kali (`192.168.12.100`) ke Target Node (`192.168.12.5`) terdeteksi di Sguil sebagai event **"PBL - ICMP Traffic Detected"** (SID 1000001, custom rule kelompok sendiri) dan **"GPL ICMP_INFO PING"**.
- Nmap scan dari Kali ke Target Node terdeteksi: event **"PBL TEST - SYN Scan Detected"** dan beberapa **"ET SCAN Suspicious inbound"** pada port 445, 1433, 1521, 3306, 5432, 5904, 5810.
- Bukti: `docs/blueteam/evidence/sguil-detection-evidence.png`, `ip-addr-security-onion-brief.png`, `ip-addr-security-onion-full.png`.
