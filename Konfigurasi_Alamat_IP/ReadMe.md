# 🔧 Konfigurasi Alamat IP pada Router Cisco

Dokumentasi ini menjelaskan langkah-langkah dasar untuk melakukan konfigurasi alamat IP pada interface router Cisco menggunakan **Command Line Interface (CLI)**.

Praktik ini dapat dilakukan pada simulator seperti **Cisco Packet Tracer**, **GNS3**, maupun pada perangkat Cisco sungguhan.

---

## 🎯 Tujuan Praktikum

- Memahami mode-mode CLI pada router
- Memberikan IP address pada interface
- Mengaktifkan interface
- Mengecek status konfigurasi
- Menyimpan konfigurasi
- Mengatur IP pada PC
- Melakukan pengujian konektivitas

---

## 🛠 Persiapan

Sebelum memulai, pastikan:

- Router terhubung menggunakan kabel console atau akses langsung CLI
- Mengetahui interface yang akan dikonfigurasi (Fa0/0, Gi0/0, dll.)
- Sudah menentukan IP Address dan subnet mask
- Topologi jaringan sudah disiapkan

---

<img width="1133" height="518" alt="Screenshot from 2026-02-03 20-41-11" src="https://github.com/user-attachments/assets/f3f29cdb-a7d3-46dc-8f63-d07b74065bf9" />


## ⚙️ Langkah Konfigurasi

### 1. Masuk ke Router

Masuk ke router hingga muncul prompt:

`rouetr>`


---

### 2. Privileged EXEC Mode

Masuk dengan:

`enable` atau `en`


---

### 3. Global Configuration Mode

Ketik:

`configure terminal` atau `conf t`

Prompt menjadi:

`Router(config)#`


---

### 4. Interface Configuration Mode

Masuk ke interface yang ingin dikonfigurasi:

`interface fa0/0` atau `inf fa0/0` ==> untuk port fastEthernet bisa berubah tergantung menggunakan port apa

Jika berhasil mode akan berubah menjadi:

`Router(config-if)#`


---

### 5. Memberikan IP Address

Contoh:

ip address 15.255.255.254 255.0.0.0

dengan perintah:

`Router(config-if)# ip address 15.255.255.254 255.0.0.0`

Pastikan subnet mask sesuai dengan jaringan.

---

### 6. Memberi Deskripsi (Opsional)

Gunakan deskripsi agar interface mudah dikenali:

`description ## ke WS1 ##`

seperti pada contoh di atas, ada bisa juga menggunakan deskripsi lain sesuai dengan kebutuhan 


---

### 7. Mengaktifkan Interface

Secara default interface dalam kondisi mati, aktifkan dengan:

`no shutdown` atau lebih ringkas menggunakan `no shut`


---

### 8. Verifikasi Konfigurasi

Keluar ke EXEC mode:

`end` agar tidak perlu penggukan perintah `exit` berulang

Cek status dengan:

`show ip interface brief` atau versi ringkasnya dengan `sh ip int brief`

Note: jika ingin tetap menjalankan perintah show ip di mode konfigurasi gunakan 'do' agar tidak perlu pindah konfigurasi exec dan global berulangkali contohnya `do sh ip int brief`


Pastikan status **up/up** dan IP sudah benar.

---

### 9. Konfigurasi Interface Lain

Ulangi langkah di atas untuk interface lainnya.

⚠️ **Perhatikan subnetting!**  
Sesuaikan prefix jaringan agar tidak terjadi overlap IP.

---

### 10. Melihat dan Menyimpan Konfigurasi

Lihat konfigurasi:


Pastikan status **up/up** dan IP sudah benar.

---

### 9. Konfigurasi Interface Lain

Ulangi langkah di atas untuk interface lainnya.

⚠️ **Perhatikan subnetting!**  
Sesuaikan prefix jaringan agar tidak terjadi overlap IP.

---

### 10. Melihat dan Menyimpan Konfigurasi

Lihat konfigurasi:
cara ini bisa di gunakan di exac mode (router#) saja tetapi jika ada berada di mode global (config#) dengan menambahkan "do"

`show running-config` atau `sh run`

Simpan agar tidak hilang setelah restart:
cara ini bisa di gunakan di exac mode (router#)

`copy running-config startup-config` atau `write` atau lebih ringkas dengan `wr`


---

### 11. Konfigurasi IP pada PC

Atur IP di setiap PC:

- IP Address: sesuai jaringan
- Subnet Mask: sesuai
- Default Gateway: IP interface router

---

### 12. Pengujian Koneksi

Lakukan ping:

`ping <alamat-ip-tujuan>`

<img width="834" height="730" alt="Screenshot from 2026-02-03 20-38-30" src="https://github.com/user-attachments/assets/f35f4572-06cd-424a-b9c7-8631f62bf07c" />


Jika reply diterima, konfigurasi berhasil ✅

---

## 🧪 Troubleshooting Dasar

Jika koneksi gagal, periksa:

- Interface sudah aktif (`no shutdown`)?
- Status `up/up`?
- IP dan subnet sesuai?
- Default gateway benar?
- Kabel terhubung?

Gunakan perintah:

`show ip route
show interfaces`


---

## 📚 Catatan

Dokumen ini merupakan bagian dari dokumentasi pembelajaran jaringan komputer.

Materi akan terus berkembang mencakup VLAN, Routing, NAT, dan Network Security.

---

✍️ *Ditulis sebagai bagian dari Computer Networking Learning Journey.*











