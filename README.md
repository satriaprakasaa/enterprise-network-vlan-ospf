# Multi-Department Enterprise Network with VLAN Segmentation and OSPF Redundancy

Sebuah simulasi arsitektur jaringan skala Enterprise yang dirancang menggunakan **Cisco Packet Tracer**. Proyek ini mengimplementasikan segmentasi departemen menggunakan VLAN, alokasi IP efisien dengan VLSM, Inter-VLAN Routing (SVI), dan High Availability (Redundancy) pada jalur tulang punggung (backbone) menggunakan protokol *dynamic routing* OSPF.

<div align="center">
  <img src="topologi.png" alt="Topologi Jaringan" width="800">
  <p><i>Topologi fisik jaringan Enterprise yang terdiri dari 2 Router Distribusi (R1, R2), 1 Multilayer Switch sebagai Core (SW-Core), 2 Access Switch, dan titik akhir (PC/Server) yang tersebar di berbagai VLAN.</i></p>
</div>

## 📌 Fitur Utama & Teknologi yang Digunakan
- **VLAN & 802.1Q Trunking:** Isolasi *broadcast domain* antar departemen (IT, HRD, Finance, Server Farm, Management) untuk keamanan dan efisiensi lalu lintas data.
- **VLSM (Variable Length Subnet Mask):** Pembagian blok IP Address secara presisi untuk menghemat ketersediaan IP.
- **SVI (Switch Virtual Interface):** *Inter-VLAN Routing* menggunakan Multilayer Switch (Layer 3) sebagai Gateway berkinerja tinggi tanpa membebani Router.
- **OSPF (Open Shortest Path First):** *Dynamic Routing* yang dikonfigurasi pada SW-Core, R1, dan R2 untuk sinkronisasi rute otomatis dan menyediakan jalur cadangan (Failover).

## 🗂 Tabel Alokasi IP Address (VLSM)

**1. Jaringan Internal (VLAN)**
| VLAN ID | Departemen | Network Address | Subnet Mask | Gateway (SVI) | Usable IP Range |
|:---:|---|---|---|---|---|
| 10 | IT | 192.168.10.0/26 | 255.255.255.192 | 192.168.10.1 | .2 - .62 |
| 20 | HRD | 192.168.10.64/27 | 255.255.255.224 | 192.168.10.65 | .66 - .94 |
| 30 | Finance | 192.168.10.96/27 | 255.255.255.224 | 192.168.10.97 | .98 - .126 |
| 40 | Server Farm | 192.168.10.128/28 | 255.255.255.240 | 192.168.10.129 | .130 - .142 |
| 99 | Management (Native)| 192.168.10.144/29 | 255.255.255.248 | 192.168.10.145 | .146 - .150 |

**2. Jaringan Point-to-Point (Backbone /30)**
| Jalur | Network Address | Subnet Mask | IP Antarmuka |
|---|---|---|---|
| SW-Core ↔ R1 | 10.10.10.0/30 | 255.255.255.252 | SW-Core: .1 \| R1: .2 |
| SW-Core ↔ R2 | 10.10.10.4/30 | 255.255.255.252 | SW-Core: .5 \| R2: .6 |
| R1 ↔ R2 | 10.10.10.8/30 | 255.255.255.252 | R1: .9 \| R2: .10 |

## 🚀 Pengujian & Verifikasi

### 1. Verifikasi Database VLAN
Membuktikan bahwa segmentasi logic untuk setiap departemen telah terdaftar dan aktif pada perangkat Switch.

<img src="vlan-database.png" alt="Database VLAN" width="600">
<br><i>Output perintah `show vlan brief` pada Switch yang menampilkan status aktif untuk VLAN 10, 20, 30, 40, dan 99.</i>

### 2. Inter-VLAN Routing (SVI)
Pengujian konektivitas lintas VLAN membuktikan bahwa Gateway pada Multilayer Switch berfungsi menjembatani jaringan departemen yang berbeda.

<img src="ping-intervlan.png" alt="Ping Inter-VLAN" width="600">
<br><i>PC dari VLAN 10 (IT) berhasil melakukan PING (Reply) ke PC di VLAN 20 (HRD) dan Server, menandakan paket berhasil dirutekan melewati SVI.</i>

### 3. Sinkronisasi Tabel Routing (OSPF)
Protokol OSPF diaktifkan di area 0 (Backbone). Router di *layer* distribusi berhasil mempelajari rute ke seluruh blok IP VLAN di lantai bawah secara dinamis.

<img src="ip-route-ospf.png" alt="Routing Table OSPF" width="600">
<br><i>Tabel `show ip route` pada R1. Kode `O` menunjukkan Router telah mengenali rute spesifik menuju VLAN 10, 20, 30, dan 40 melalui Gateway SW-Core.</i>

### 4. Redundancy & Failover Test (Stress Test)
Skenario ini membuktikan arsitektur *High Availability*. Ping tanpa henti (`ping -t`) dikirimkan melewati jalur utama menuju Router cadangan. Saat port jalur utama dimatikan (`shutdown`), OSPF merespons perubahan topologi secara instan.

<img src="failover-test.png" alt="Uji Failover" width="600">
<br><i>Ping mengalami Request Timed Out (RTO) sesaat saat kabel utama terputus, namun algoritma OSPF langsung membelokkan lalu lintas data ke jalur cadangan (via R1) sehingga ping kembali Reply tanpa intervensi manual.</i>

## 🛠 Cara Penggunaan (How to Use)
1. Unduh file `Enterprise-Network-Failover.pkt` dari repositori ini.
2. Buka menggunakan **Cisco Packet Tracer** (Rekomendasi: Versi 9.0.1 atau lebih baru).
3. Kata sandi mode `enable` (jika diminta): `admin` *(Ganti jika Anda memasang password di konfigurasi)*.
4. Lakukan eksperimen Anda sendiri dengan mematikan port pada SW-Core untuk melihat algoritma OSPF bekerja secara otomatis.
