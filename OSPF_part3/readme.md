# 🌐 Konfigurasi OSPF Part 3: Serial Cable & Network Types

Dokumentasi ini membahas konfigurasi OSPF pada interface Serial, penanganan masalah perbedaan *Network Type*, dan cara menghubungkan jaringan OSPF ke internet (External Route).

---

<img width="1349" height="563" alt="ospf_part_3" src="https://github.com/user-attachments/assets/c52705ba-4e72-4cfc-a9bb-71d136082ef7" />


## 🏗️ 1. Konfigurasi Kabel Serial (DCE vs DTE)

Berbeda dengan kabel Ethernet, kabel Serial memiliki dua sisi: **DCE (Data Communications Equipment)** dan **DTE (Data Terminal Equipment)**. Sisi DCE bertanggung jawab untuk mengatur kecepatan data melalui perintah `clock rate`.

### Memeriksa Jenis Kabel
Gunakan perintah berikut untuk menentukan router mana yang bertindak sebagai DCE:

```bash
Router# show controllers serial 2/0
```
<img width="767" height="285" alt="R1 sebagai DCE" src="https://github.com/user-attachments/assets/63d4a7cf-8445-4239-b0e6-afa9185b3a1d" />


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

<img width="576" height="358" alt="beda network type R4" src="https://github.com/user-attachments/assets/53c5179c-1c87-4b54-b5cb-0f4869d87866" />

(gambar network type broadcast)

<img width="579" height="239" alt="network type boardcast" src="https://github.com/user-attachments/assets/3929e9cd-9fad-44e6-bf10-f08a0fa0f5da" />

(gambar network type ptp)

<img width="579" height="239" alt="network type ptp" src="https://github.com/user-attachments/assets/6a2871fe-c737-4530-9670-31861804c0ac" />

Solusi
OSPF tidak bisa mentoleransi perbedaan Network Type dalam satu segmen kabel yang sama karena cara mereka mengiklankan LSA (Link State Advertisement) berbeda. Kita harus menyamakannya ke default (Broadcast).

Perbaikan di R3:
```
R3(config-if)# interface g1/0
R3(config-if)# no ip ospf network point-to-point
```

<img width="578" height="446" alt="R4 network type sama" src="https://github.com/user-attachments/assets/f6995db1-7e21-4005-b574-71f86478ba03" />


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

<img width="578" height="212" alt="ping eksternal gagal" src="https://github.com/user-attachments/assets/2f82df17-4c99-4274-b5dc-aa1d70db2a3f" />


Kesimpulan: OSPF Part 3 mengajarkan kita bahwa kedekatan fisik (Layer 1) dan status neighbor (Layer 3) saja tidak cukup. 
Parameter seperti Network Type harus konsisten agar pertukaran data bisa berjalan sempurna.




