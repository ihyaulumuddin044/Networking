# 🌐 Konfigurasi OSPF Part 3: Serial Cable & Network Types

Dokumentasi ini membahas konfigurasi OSPF pada interface Serial, penanganan masalah perbedaan *Network Type*, dan cara menghubungkan jaringan OSPF ke internet (External Route).

---

## 🏗️ 1. Konfigurasi Kabel Serial (DCE vs DTE)

Berbeda dengan kabel Ethernet, kabel Serial memiliki dua sisi: **DCE (Data Communications Equipment)** dan **DTE (Data Terminal Equipment)**. Sisi DCE bertanggung jawab untuk mengatur kecepatan data melalui perintah `clock rate`.

### Memeriksa Jenis Kabel
Gunakan perintah berikut untuk menentukan router mana yang bertindak sebagai DCE:

```bash
Router# show controllers serial 2/0
```
(gambar R1 sebagai DCE)

Implementasi Konfigurasi
Jika router terdeteksi sebagai DCE, kita wajib menentukan clock rate. Di sini kita menggunakan kecepatan 128.000.

Konfigurasi di R1 (DCE):
```
R1(config)# interface serial 2/0
R1(config-if)# clock rate 128000
R1(config-if)# ip address 192.168.12.1 255.255.255.252
R1(config-if)# no shutdown
```

Konfigurasi di R2 (DTE):

```
R2(config)# interface serial 2/0
R2(config-if)# ip address 192.168.12.2 255.255.255.252
R2(config-if)# no shutdown
```

---

## ⚠️ 2. Masalah Network Type Mismatch
Pernahkah Anda melihat status OSPF FULL tapi rute tidak muncul di tabel routing? Salah satu penyebab utamanya adalah perbedaan Network Type.

Gejala Masalah
Pada kasus ini, R3 menggunakan tipe Point-to-Point (PtP) sementara R4 menggunakan tipe Broadcast.

Status Neighbor: FULL (Kedekatan terjalin).

Routing Table: Rute dari tetangga tidak muncul.

Hasil Ping: Reply from 10.0.2.254: Destination host unreachable.

(gambar beda network type R4)
(gambar network type broadcast)
(gambar network type ptp)

Solusi
OSPF tidak bisa mentoleransi perbedaan Network Type dalam satu segmen kabel yang sama karena cara mereka mengiklankan LSA (Link State Advertisement) berbeda. Kita harus menyamakannya ke default (Broadcast).

Perbaikan di R3:
```
R3(config-if)# interface g1/0
R3(config-if)# no ip ospf network point-to-point
```

(gambar R4 network type sama)

Setelah disamakan, tabel routing pada R4 akan segera terisi dan koneksi antar jaringan kembali normal.

---

## 🌍 3. Menghubungkan ke Jaringan Eksternal (Internet)
Untuk membuat seluruh jaringan OSPF bisa mengakses internet, kita perlu mengonfigurasi Static Default Route pada router yang terhubung ke ISP dan mengiklankannya ke dalam area OSPF. Router ini akan bertindak sebagai ASBR (Autonomous System Boundary Router).

Langkah-langkah:
Buat rute statis menuju IP Gateway ISP.

Gunakan perintah default-information originate agar router OSPF lain mengetahui jalan keluar menuju internet.

Konfigurasi di R5 (ASBR):
```
R5(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.2
R5(config)# router ospf 1
R5(config-router)# default-information originate
```

Analisis Hasil Ping
Jika Anda melakukan ping ke 8.8.8.8 dan hasilnya Request Time Out (RTO), jangan panik!

* Penyebab: Router ISP mungkin menerima paket Anda, tetapi tidak tahu jalan balik (Return Route) ke jaringan lokal Anda (10.0.0.0/24).

* Solusi: Di dunia nyata, kita akan menggunakan NAT (Network Address Translation) untuk menyamarkan IP lokal menjadi IP publik, atau menambahkan rute balik di sisi ISP.

(gambar ping eksternal gagal)

Kesimpulan: OSPF Part 3 mengajarkan kita bahwa kedekatan fisik (Layer 1) dan status neighbor (Layer 3) saja tidak cukup. 
Parameter seperti Network Type harus konsisten agar pertukaran data bisa berjalan sempurna.




