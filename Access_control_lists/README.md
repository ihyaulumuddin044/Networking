# Panduan Konfigurasi Standard ACLs (Access Control Lists)

Dokumentasi ini berisi panduan komprehensif mengenai konsep dasar, cara kerja, analisis masalah (*troubleshooting*), serta praktik langsung (*hands-on*) penerapan Standard ACL (baik *Numbered* maupun *Named*) pada perangkat Cisco Jaringan.

---

## 1. Konsep Dasar Standard ACL

**Standard ACL (Access Control List)** adalah aturan pada jaringan komputer yang berfungsi untuk memfilter dan mengontrol lalu lintas data berdasarkan alamat IP sumber (*Source IP*). Jenis ACL ini hanya melihat dari mana paket data tersebut berasal, tanpa memperhatikan tujuan akhir, protokol (seperti TCP/UDP), atau nomor port.

### Karakteristik & Cara Kerja
* **Cara Kerja:** Jika IP sumber cocok dengan aturan yang dibuat, router akan mengizinkan (*permit*) atau menolak (*deny*) paket data tersebut untuk diteruskan.
* **Nomor Pengenal (ID)
:** Pada perangkat Cisco, Standard ACL diidentifikasi menggunakan nomor **1 hingga 99** (dan tambahan rentang **1300 hingga 1999** untuk cakupan yang lebih besar).
* **Lokasi Penerapan:** Standar ACL biasanya diterapkan **paling dekat dengan alamat tujuan (*destination*)**, bukan di dekat perangkat sumber. Hal ini dilakukan untuk mencegah pemblokiran salah sasaran di tengah jalur jaringan (*placement optimization*).
* **Perbedaan dengan Extended ACL:** Jika Standard ACL hanya memfilter berdasarkan IP asal, Extended ACL memberikan kontrol yang jauh lebih spesifik karena mampu memeriksa IP tujuan, jenis protokol (seperti HTTP, FTP, DNS), serta nomor port (misal port 80, 443, 22).

---

## 2. Topologi Jaringan & Skenario Lab

Penerapan ini disimulasikan menggunakan Cisco Packet Tracer dengan arsitektur jaringan yang melibatkan Router 1 (R1), Router 2 (R2), beberapa PC Client, dan Server.

<img width="879" height="362" alt="standard_acl_topologi" src="https://github.com/user-attachments/assets/8fc70f5a-87bf-4edd-b6ae-9fc299b168a1" />

---


## Tahap 1: Konfigurasi OSPF pada R1 dan R2

Sebelum menerapkan filter keamanan (ACL), semua segmen jaringan harus dipastikan saling terhubung menggunakan protokol routing dinamis OSPF Area 0.

### a. Konfigurasi OSPF pada R1
```routing
R1(config)# router ospf 1
R1(config-router)# network 172.16.0.0 0.0.255.255 area 0
R1(config-router)# network 203.0.113.0 0.0.0.3 area 0
```
Analisis Perintah:

* router ospf 1: Menginisiasi proses protokol routing OSPF dengan ID Proses 1.

* network 172.16.0.0 0.0.255.255 area 0: Mendaftarkan network address. Wildcard mask 0.0.255.255 berarti router akan mencocokkan secara presisi dua oktet pertama (172.16.x.x). Interface apa pun di R1 yang memiliki IP berawalan 172.16 otomatis menjalankan OSPF di Area 0 (Backbone Area).

* network 203.0.113.0 0.0.0.3 area 0: Mendaftarkan segmen point-to-point (kabel serial/LAN) yang menghubungkan R1 langsung ke R2.

⚠️ Catatan Penting Keamanan (NIST SP 800-53):
Penggunaan wildcard mask berskala luas seperti 0.0.255.255 tidak disarankan di lingkungan produksi nyata. Hal ini bertentangan dengan kontrol AC-6 (Least Privilege) dalam NIST SP 800-53 yang menegaskan bahwa sistem atau pengguna hanya boleh diberikan hak akses seminimal mungkin. Di dunia nyata, daftarkan IP secara spesifik per subnet interface (misal: 0.0.0.255).

Untuk memverifikasi status interface OSPF pada R1, gunakan perintah berikut:
`R1# show ip ospf interface`
Pastikan status pada setiap interface yang terdaftar sudah menunjukkan status "up".

### b. Konfigurasi OSPF pada R2

```
R2(config)# router ospf 1
R2(config-router)# network 192.168.0.0 0.0.255.255 area 0
R2(config-router)# network 203.0.113.0 0.0.0.3 area 0
```

### c. Pengujian & Verifikasi OSPF Neighbor
Periksa apakah hubungan ketetanggaan (adjacency) OSPF antara R1 dan R2 telah terbentuk dengan benar:

```
R2# show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
203.0.113.1       1   FULL/DR         00:00:39    203.0.113.1     GigabitEthernet0/0
```

### Analisis Status:
Status FULL menandakan bahwa pertukaran data link-state antar-router telah selesai dan sinkron sepenuhnya. Status DR (Designated Router) pada tetangga menunjukkan bahwa R1 bertindak sebagai ketua/koordinator utama dalam protokol OSPF di segmen tersebut.

Lakukan pengujian koneksi dasar menggunakan PING (ICMP) atau Simple PDU dari PC ke Server untuk memastikan rute komunikasi sukses sebelum ACL dipasang.

## Tahap 2: Implementasi Kebijakan Jaringan Menggunakan ACL
Kebijakan Jaringan (Network Policies):
* Hanya PC 1 dan PC 3 yang diizinkan mengakses Server 1 di jaringan 192.168.1.0/24.

* Host di jaringan 172.16.2.0/24 tidak boleh mengakses jaringan 192.168.2.0/24.

* Jaringan 172.16.1.0/24 tidak boleh mengakses jaringan 172.16.2.0/24.

* Jaringan 172.16.2.0/24 tidak boleh mengakses jaringan 172.16.1.0/24.

Kasus A: Batasan Akses ke Jaringan Server 1 (Named Standard ACL)
Aturan pertama diimplementasikan menggunakan Standard Named ACL pada R2.

### Konfigurasi Awal (Salah Letak / Misconfiguration)

```
R2(config)# ip access-list standard TO_192.168.1.0/24
R2(config-std-nacl)# permit 172.16.1.1
R2(config-std-nacl)# permit 172.16.2.1
R2(config-std-nacl)# deny any
R2(config-std-nacl)# interface g0/0
R2(config-if)# ip access-group TO_192.168.1.0/24 out
```

### Analisis Masalah & Troubleshooting
Ketika pengujian ping dilakukan dari PC1 ke Server, hasil yang didapat adalah Request Timed Out (RTO) secara massal:

```
C:\> ping 192.168.1.100
Pinging 192.168.1.100 with 32 bytes of data:
Request timed out.
Request timed out.
```

### Mengapa RTO, bukan Destination Host Unreachable?
Kesalahan terjadi pada penempatan access-group. Interface g0/0 pada R2 adalah interface yang mengarah ke arah R1 (jalur antar-router). Ketika paket balasan (reply) kembali dari Server menuju PC, paket tersebut melewati g0/0 sebagai arah outbound. Karena IP asal paket balasan adalah IP Server (bukan IP PC yang di-permit), maka aturan implisit deny any pada ACL langsung memblokir paket balasan tersebut. Akibatnya, paket permintaan sampai ke server, namun paket balasannya dibuang di R2, menghasilkan efek RTO.

### Solusi & Perbaikan Konfigurasi
ACL harus dipindahkan ke interface terdekat dengan tujuan, yaitu g0/2 (gateway yang langsung mengarah ke segmen 192.168.1.0/24).

```
R2(config)# interface g0/0
R2(config-if)# no ip access-group TO_192.168.1.0/24 out
R2(config-if)# interface g0/2
R2(config-if)# ip access-group TO_192.168.1.0/24 out
```

Hasil Uji Coba Setelah Perbaikan:
PC 1 (Diizinkan):

```
C:\> ping 192.168.1.100
Reply from 192.168.1.100: bytes=32 time=1ms TTL=126
```

PC 2 (Ditolak / Tidak Terdaftar):

```
DOS
C:\> ping 192.168.1.100
Reply from 203.0.113.2: Destination host unreachable.
```
(Sistem secara benar memberikan respons Destination host unreachable karena diblokir langsung oleh aturan inbound/outbound router gateway).

### Kasus B: Blokir Akses Segmen 172.16.2.0/24 ke Jaringan 192.168.2.0/24
Aturan ini dikonfigurasi menggunakan Named Standard ACL pada router tujuan (R2), diterapkan pada interface g0/1 arah outbound (arah ke jaringan 192.168.2.0/24).

Cuplikan kode
```
R2(config)# ip access-list standard TO_192.168.2.0/24
R2(config-std-nacl)# deny 172.16.2.0 0.0.0.255
R2(config-std-nacl)# permit any
R2(config-std-nacl)# interface g0/1
R2(config-if)# ip access-group TO_192.168.2.0/24 out
```

Hasil Uji Coba:
Uji Coba PC 1 (Subnet 172.16.1.0/24 - Harus Berhasil):

```
C:\> ping 192.168.2.100
Reply from 192.168.2.100: bytes=32 time<1ms TTL=126
```
Uji Coba PC 2 (Subnet 172.16.2.0/24 - Harus Diblokir):

```
C:\> ping 192.168.2.100
Reply from 203.0.113.2: Destination host unreachable.
```
Konfigurasi berhasil memblokir traffic dari rentang alamat 172.16.2.0/24 secara akurat.

### Kasus C: Blokir Akses Antar-Segmen LAN Internal (Numbered ACL)
Untuk membatasi komunikasi antar-subnet internal (172.16.1.0/24 dan 172.16.2.0/24), kita menggunakan Numbered Standard ACL pada R1.

Cuplikan kode
```
R1(config)# access-list 1 deny 172.16.1.0 0.0.0.255
R1(config)# access-list 1 permit any
R1(config)# access-list 2 deny 172.16.2.0 0.0.0.255
R1(config)# access-list 2 permit any

R1(config)# interface g0/2
R1(config-if)# ip access-group 1 out

R1(config)# interface g0/1
R1(config-if)# ip access-group 2 out
```
Analisis Konfigurasi:

access-list 1 memblokir asal 172.16.1.0/24 dan dipasang pada interface g0/2 (outbound menuju LAN 172.16.2.0/24).

access-list 2 memblokir asal 172.16.2.0/24 dan dipasang pada interface g0/1 (outbound menuju LAN 172.16.1.0/24).

Tugas Mandiri Lab / Validasi Akhir:
Silakan lakukan pengujian ping timbal balik dari host di IP 172.16.1.1 menuju 172.16.2.1 dan sebaliknya. Status akhir yang diharapkan adalah kedua arah menghasilkan respons Destination host unreachable, membuktikan isolasi keamanan antar LAN internal telah berhasil dikonfigurasi.

Kesimpulan: Standard ACL sangat efektif untuk pemfilteran lalu lintas dasar berbasis IP asal. Untuk kendali yang lebih granular, pertimbangkan penggunaan Extended ACL.























