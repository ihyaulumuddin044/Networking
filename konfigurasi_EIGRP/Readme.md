# 🌐 Panduan Konfigurasi EIGRP (Enhanced Interior Gateway Routing Protocol)

Repositori ini berisi panduan praktis mengenai definisi, cara kerja, dan langkah-langkah konfigurasi **EIGRP** pada perangkat Cisco. Dokumentasi ini disusun untuk mempermudah pemahaman konsep *hybrid routing* dan implementasi teknisnya di lapangan.

---

## 📖 1. Apa itu EIGRP?

**EIGRP** adalah protokol routing **"hybrid"** (campuran) yang dikembangkan oleh Cisco. Protokol ini menggabungkan kecepatan protokol *link-state* dengan kemudahan konfigurasi *distance vector*.

* 🚀 **Konvergensi Cepat:** Mampu menemukan jalur alternatif dalam hitungan detik.
* ⚖️ **Efisien:** Menggunakan sumber daya CPU dan bandwidth yang minim karena hanya mengirim update saat ada perubahan.
* 🛠️ **Mudah Dikelola:** Pengaturan yang relatif simpel namun sangat powerful untuk skala Enterprise.



---

## ⚙️ 2. Cara Kerja (Algoritma DUAL)

EIGRP menggunakan algoritma **DUAL (Diffusing Update Algorithm)** untuk menghitung jalur terpendek dan menjamin jaringan bebas dari *looping* (loop-free).

### 📊 Metrik Penghitungan (K-Values)
EIGRP tidak hanya melihat jarak, tapi juga kualitas jalur:
| Komponen | Deskripsi |
| :--- | :--- |
| ⚡ **Bandwidth** | Kecepatan link terkecil dalam jalur. |
| ⏱️ **Delay** | Total waktu tempuh paket (dalam mikrodetik). |
| 🛡️ **Reliability** | Stabilitas atau tingkat keandalan link (1-255). |
| 📈 **Load** | Kepadatan lalu lintas pada link (1-255). |

### 🗂️ Tiga Tabel Utama EIGRP
1.  **Neighbor Table**: Daftar router tetangga yang terhubung langsung.
2.  **Topology Table**: Daftar seluruh jalur yang dipelajari (berisi *Successor* & *Feasible Successor*).
3.  **Routing Table**: Daftar jalur terbaik (**Successor**) yang aktif digunakan untuk mengirim data.

---

## 🌟 3. Keunggulan Utama
* 🔄 **Instant Backup**: Jika jalur utama putus, EIGRP langsung beralih ke jalur cadangan tanpa kalkulasi ulang yang lama.
* 📡 **Partial Updates**: Tidak mengirim seluruh tabel routing secara periodik; hanya mengirim bagian yang berubah.
* 🔢 **Classless Support**: Mendukung penuh **VLSM** dan **CIDR**.
* 🔀 **Unequal Cost Load Balancing**: Fitur unik yang memungkinkan pembagian trafik pada jalur dengan kecepatan berbeda.

---

## 🛠️ 4. Langkah-Langkah Konfigurasi

### 📍 Tahap 1: Persiapan Topologi
Lakukan konfigurasi alamat IP pada setiap interface fisik (FastEthernet/GigabitEthernet) sesuai perencanaan.  
**(gambar topologi EIGRP)**
<img width="872" height="444" alt="topologi EIGRP" src="https://github.com/user-attachments/assets/af39a5ba-216b-41fc-bbfd-906522d244aa" />


### ☁️ Tahap 2: Konfigurasi Interface Loopback
*Interface Loopback* digunakan sebagai identitas router (Router ID) karena sifatnya yang virtual dan tidak pernah mati (selama router menyala).

| Router | Interface | IP Address | Subnet Mask |
| :--- | :--- | :--- | :--- |
| **R1** | Loopback0 | `1.1.1.1` | `255.255.255.255` |
| **R2** | Loopback0 | `2.2.2.2` | `255.255.255.255` |
| **R3** | Loopback0 | `3.3.3.3` | `255.255.255.255` |
| **R4** | Loopback0 | `4.4.4.4` | `255.255.255.255` |

**Contoh Perintah pada R1:**
```
bash
R1(config)# interface loopback 0
R1(config-if)# ip address 1.1.1.1 255.255.255.255
```

gambar konfigurasi:

| Router | Interface & IP Address |
| :--- | :--- |
| **R1** | <img src="https://github.com/user-attachments/assets/9bdfaa8b-442f-45b2-9e1d-ff84a153c3a1" width="400"> |
| **R2** | <img src="https://github.com/user-attachments/assets/3baa302d-acec-43b2-9027-1d8500a6cf34" width="400"> |
| **R3** | <img src="https://github.com/user-attachments/assets/874604b4-6ebf-43cd-8680-9d34c13b9781" width="400"> |
| **R4** | <img src="https://github.com/user-attachments/assets/5db6ea48-7dfd-4696-bdf9-2aa537012ff5" width="400"> |


---

## 🚀 Tahap 3: Aktivasi Protokol EIGRP
⚡ A. Konfigurasi R4 (Metode Wildcard Luas)
Menggunakan perintah "sapu jagat" untuk mengaktifkan EIGRP di semua interface secara otomatis.

```
R4(config)# router eigrp 100
R4(config-router)# network 0.0.0.0 255.255.255.255
R4(config-router)# no auto-summary
R4(config-router)# passive-interface g2/0
R4(config-router)# passive-interface l0
```

## 🎯 B. Konfigurasi R1, R2, R3 (Metode Spesifik)
Metode yang lebih presisi dan aman untuk jaringan nyata.

```
R1(config)# router eigrp 100
R1(config-router)# network 10.0.12.0 0.0.0.3
R1(config-router)# network 10.0.13.0 0.0.0.3
R1(config-router)# network 1.1.1.1 0.0.0.0
R1(config-router)# passive-interface loopback 0
R1(config-router)# no auto-summary
```

[!IMPORTANT]
AS Number (100): Harus sama di seluruh router agar bisa saling bertukar informasi.
No Auto-Summary: Menghindari peringkasan IP otomatis ke Classful (A/B/C).
Passive Interface: Menjaga efisiensi dengan tidak mengirim paket EIGRP ke arah PC atau Loopback.

---
### 🔍 5. Verifikasi & Pengujian
Gunakan perintah berikut untuk memastikan jaringan sudah stabil:

* show ip eigrp neighbors 🤝 : Pastikan semua tetangga muncul di daftar.
* show ip route eigrp 🛣️ : Jalur yang dipelajari akan ditandai dengan kode D.
* show ip protocols 📜 : Cek AS Number dan network yang terdaftar.

## 🧪 Uji Konektivitas

<img width="797" height="646" alt="uji koneksi" src="https://github.com/user-attachments/assets/c862b6da-fe8c-4780-9ff1-9b23d6a52a7d" />

hasil dari traceroute dari pc ke router tujuan


Analisis Tracert:
Gunakan perintah traceroute (atau tracert di Windows) untuk melihat lompatan paket. Jika konfigurasi benar, paket akan memilih jalur terbaik yang dihitung oleh algoritma DUAL.














