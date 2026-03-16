# Dokumentasi Implementasi: First Hop Redundancy Protocol (FHRP) dengan HSRP

## 1. Pendahuluan
**First Hop Redundancy Protocol (FHRP)** adalah kategori protokol jaringan yang menyediakan redundansi *default gateway* dengan mengizinkan dua atau lebih router/switch Layer 3 bekerja sama berbagi satu alamat IP/MAC Virtual.

* **Tujuan:** Meningkatkan ketersediaan (*high availability*) jaringan dengan menghilangkan *single point of failure* pada gateway.
* **Cara Kerja:** Router-router dalam grup FHRP berbagi satu **Virtual IP (VIP)** yang dikonfigurasi sebagai gateway pada perangkat host.
* **Failover:** Jika router Utama (*Active*) gagal, router Cadangan (*Standby*) mengambil alih secara otomatis.

### Jenis-jenis FHRP Umum
| Protokol | Pengembang | Fitur Utama |
| :--- | :--- | :--- |
| **HSRP** | Cisco Proprietary | Menggunakan peran *Active* dan *Standby*. |
| **VRRP** | Open Standard | Standar terbuka (IEEE), sangat mirip dengan HSRP. |
| **GLBP** | Cisco Proprietary | Mendukung Redundansi sekaligus *Load Balancing*. |

---

## Implementasi FHRP Menggunakan HSRP

Pada praktik ini kita akan mengkonfigurasi HSRP pada topologi berikut.

<img width="1130" height="436" alt="Toplogi Hsrp" src="https://github.com/user-attachments/assets/106ef407-8a60-4ec4-829b-4959086bc19d" />


Untuk penjelasan teknis mengenai cara kerja HSRP secara lebih mendalam, Anda dapat merujuk ke dokumentasi resmi Cisco atau sumber lain seperti Wikipedia.
---

## 1. Konfigurasi HSRP pada Router
   
=== Konfigurasi pada R1 ===

Pada R1 kita akan mengkonfigurasinya sebagai router utama (Active Router).

```
R1(config)# interface g0/0
R1(config-if)# standby version 2
R1(config-if)# standby 1 ip 10.0.1.254
R1(config-if)# standby 1 priority 100
R1(config-if)# standby 1 preempt
```
Penjelasan Konfigurasi

A. standby version 2

Menentukan versi HSRP yang digunakan.

HSRP versi 2 menggunakan alamat multicast yang berbeda dibanding versi 1 dan mendukung lebih banyak group.

B. standby 1 ip 10.0.1.254

Menentukan Virtual IP Address (VIP) untuk HSRP Group 1.

Pada sisi client, alamat yang digunakan sebagai default gateway bukanlah IP asli router, melainkan IP virtual ini.

Keuntungannya:

Jika router utama mati, router cadangan akan mengambil alih VIP tersebut tanpa mengubah konfigurasi gateway pada client.

C. standby 1 priority 100

Menentukan prioritas router dalam pemilihan Active Router.

Router dengan nilai priority tertinggi akan menjadi Active Router.

Catatan:

Nilai default priority adalah 100.

D. standby 1 preempt

Memungkinkan router dengan prioritas lebih tinggi untuk merebut kembali posisi Active Router ketika router tersebut kembali aktif setelah sebelumnya mati.

Tanpa perintah ini, router yang sebelumnya menjadi Standby tetapi sudah naik menjadi Active tidak akan menyerahkan kembali peran tersebut.

=== Konfigurasi pada R2 ===
```
R2(config)# interface g0/0
R2(config-if)# standby version 2
R2(config-if)# standby 1 ip 10.0.1.254
R2(config-if)# standby 1 priority 50
```

Dengan konfigurasi ini:

* R1 menjadi Active Router

* R2 menjadi Standby Router

Verifikasi Status HSRP

Status HSRP dapat diperiksa dengan perintah:

`show standby brief`

Contoh hasil pada masing-masing router:

pada R1

<img width="539" height="102" alt="standby_br_R1" src="https://github.com/user-attachments/assets/07b49cc7-de55-4cd7-8b76-9fe30456f5be" />


pada R2

<img width="539" height="102" alt="standby_br_R2" src="https://github.com/user-attachments/assets/b5efacd3-2491-459f-9f7a-669d15a87cf4" />

=== Apa yang Terjadi Jika Versi HSRP Berbeda? ===

Jika kita menjalankan perintah berikut pada salah satu router:

Maka:

* R1 menggunakan HSRP versi 2 dan mengirim paket hello ke multicast 224.0.0.102

* R2 menggunakan HSRP versi 1 dan mendengarkan multicast 224.0.0.2

Akibatnya kedua router tidak dapat saling berkomunikasi.
R2 akan menganggap R1 tidak aktif dan mengangkat dirinya sendiri menjadi Active Router.
Hal ini menyebabkan dua router sama-sama aktif menggunakan VIP yang sama.

Dampak pada Jaringan

Kondisi ini dapat menyebabkan beberapa masalah serius:

Trafik Tidak Stabil:

Client dapat mengirim paket secara bergantian ke R1 dan R2 tergantung hasil ARP terakhir.

Koneksi Terputus:

Sesi aplikasi seperti video call, game online, atau SSH dapat terputus karena paket data berpindah-pindah jalur.

---

## 2. Konfigurasi VIP Sebagai Default Gateway pada Client

Pada perangkat end device (PC), default gateway harus diatur ke alamat VIP HSRP.

`10.0.1.254`

Dengan konfigurasi ini, client tidak perlu mengetahui router mana yang aktif.

Pengujian Konektivitas

Pengujian dilakukan menggunakan perintah:

`ping 8.8.8.8`

ping error

=== Troubleshooting ===

A. Memeriksa IP Protocol
show ip protocols

<img width="539" height="253" alt="masalah sh ip pro" src="https://github.com/user-attachments/assets/86b4fa29-4b63-472e-a8f5-e523db0854ef" />


OSPF seharusnya sudah dalam kondisi Full adjacency dan seharusnya sudah dapat melakukan ping ke 8.8.8.8 yang merupakan loopback dari R3.

B. Memeriksa Routing Table
show ip route

<img width="535" height="306" alt="table routing 1" src="https://github.com/user-attachments/assets/7955372f-7a7b-4c80-abb6-3c7f374562da" />


Terlihat pesan berikut:

Gateway of last resort is not set

Ini berarti router tidak memiliki default route menuju jaringan luar.

Ketika client mengirim paket ke 8.8.8.8, router tidak mengetahui jalur menuju tujuan tersebut sehingga menghasilkan pesan Destination host unreachable.

Konfigurasi Default Route

Untuk mengatasi masalah tersebut, kita dapat menambahkan default route dan mengiklankannya melalui OSPF.

```
R1(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.2
R1(config)# router ospf 1
R1(config-router)# default-information originate
```

Dengan konfigurasi ini:

* R1 menjadi ASBR (Autonomous System Boundary Router)

* R2 akan mempelajari default route melalui OSPF

(ping berhasil)

Setelah konfigurasi dilakukan, konektivitas jaringan kembali normal.

## 3. Analisis Jalur Next Hop
Traceroute Sebelum R1 Mati

<img width="527" height="121" alt="tracert sebelum R1 mati" src="https://github.com/user-attachments/assets/7de4ac3f-3445-4b3f-bab6-4ec6760da781" />


Traceroute Setelah R1 Mati

<img width="527" height="121" alt="tracert setelah R1 mati" src="https://github.com/user-attachments/assets/f4922240-7674-4af6-9788-b8b5c00ec732" />


Mengapa IP yang Muncul Bukan VIP?

Anda mungkin bertanya:

Mengapa alamat yang muncul adalah .253 dan .252, padahal VIP HSRP adalah .254?

Hal ini terjadi karena traceroute menggunakan TTL (Time To Live).

Ketika paket traceroute mencapai router, router akan menghentikan paket tersebut dan mengirim pesan balik ICMP Time Exceeded.

Pesan balasan ini menggunakan alamat IP asli dari interface router, bukan IP virtual HSRP.

Analogi Sederhana

Bayangkan VIP (.254) sebagai sebuah pintu masuk.

Client hanya tahu bahwa untuk keluar jaringan ia harus melewati pintu tersebut.

Namun ketika router menjawab pertanyaan traceroute, router akan menggunakan identitas aslinya.

Contoh:

* Saat R1 aktif, traceroute akan menampilkan 10.0.1.253

* Saat R1 mati, R2 mengambil alih VIP, sehingga traceroute menampilkan 10.0.1.252

Perubahan ini menjadi bukti bahwa HSRP failover berjalan dengan baik.




