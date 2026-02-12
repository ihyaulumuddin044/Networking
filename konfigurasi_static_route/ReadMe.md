# 🌐 Konfigurasi Static Route (Routing Statis)

Dokumentasi ini membahas konfigurasi **routing statis** untuk menghubungkan:

- Jaringan 192.168.1.0/24 (terhubung ke R1)
- Jaringan 192.168.3.0/24 (terhubung ke R3)

Dengan Router 2 (R2) sebagai penghubung di tengah.

<img width="457" height="187" alt="topologi" src="https://github.com/user-attachments/assets/d59fa161-5ea0-4be9-b52b-dce53accab30" />

Topologi sederhana:

R1 ↔ R2 ↔ R3

---

## 🎯 Tujuan

- Menghubungkan jaringan yang berbeda melalui static route
- Memahami konsep destination network dan next-hop
- Melakukan troubleshooting routing

---

# 1️⃣ Konfigurasi pada Router 1 (R1)

<img width="547" height="140" alt="ip address di R1" src="https://github.com/user-attachments/assets/730244f5-7074-4ed4-a9f7-ef617edbbe2a" />


Terlihat pada gambar bahwa pada port GigabitEthernet 0/0 dan 0/1 telah diberikan alamat IP.

Contoh ilustrasi konfigurasi:

`interface g0/1
ip address 192.168.1.254 255.255.255.0
no shutdown`

`interface g0/0
ip address 192.168.12.1 255.255.255.0
no shutdown`


Interface `g0/1` terhubung ke Router 2.

---

### 🔁 Menambahkan Static Route di R1

<img width="535" height="63" alt="cara rute di R1" src="https://github.com/user-attachments/assets/4de3c9c7-64d6-4054-92c4-7c1781669cf7" />


Untuk membuat jalur menuju jaringan R3, gunakan perintah:

`ip route 192.168.3.0 255.255.255.0 192.168.12.2`

Penjelasan:

1. Destination network → `192.168.3.0`
2. Subnet mask → `255.255.255.0`
3. Next-hop / forwarding route → `192.168.12.2` (IP Router 2)

---

### 🔍 Verifikasi Routing R1


<img width="543" height="267" alt="sh ip route R1" src="https://github.com/user-attachments/assets/1e5cdb41-216d-4711-b734-fb568ab7651b" />


Pastikan muncul entri:

S 192.168.3.0/24 [1/0] via 192.168.12.2


Huruf **S** menandakan static route.

---

# 2️⃣ Konfigurasi pada Router 3 (R3)

<img width="577" height="168" alt="sh ip dan ip route R2" src="https://github.com/user-attachments/assets/fb5ba453-aef0-45a9-8942-438a046a1431" />


Pada Router 3, kita juga harus membuat jalur kembali menuju jaringan R1.

Contoh konfigurasi IP:

interface g0/1
ip address 192.168.3.154 255.255.255.0
no shutdown

interface g0/0
ip address 192.168.13.3 255.255.255.0
no shutdown


---

### 🔁 Menambahkan Static Route di R3

Untuk menuju jaringan 192.168.1.0:

`ip route 192.168.1.0 255.255.255.0 192.168.13.2`


Penjelasan:

1. Destination network → `192.168.1.0`
2. Subnet mask → `255.255.255.0`
3. Forwarding route → `192.168.13.2` (IP Router 2)

---

### 🔍 Verifikasi Routing R3


<img width="546" height="270" alt="sh ip route R3" src="https://github.com/user-attachments/assets/e8a41ea2-d2a7-4f03-9571-d2d9dfbd8c57" />

Pastikan terdapat static route menuju 192.168.1.0/24.

---

# 3️⃣ Konfigurasi pada Router 2 (R2)

<img width="577" height="168" alt="sh ip dan ip route R2" src="https://github.com/user-attachments/assets/2ed9f300-ff4b-497b-94cc-e6a0b0b85c22" />


Router 2 berfungsi sebagai penghubung antara R1 dan R3.

Contoh konfigurasi:

interface g0/0
ip address 192.168.12.2 255.255.255.0
no shutdown

interface g0/1
ip address 192.168.13.2 255.255.255.0
no shutdown


---

### 🔁 Routing pada R2

<img width="599" height="97" alt="ip route R2" src="https://github.com/user-attachments/assets/1f47845a-3311-41bb-9bbf-490f63856a42" />


Karena R2 terhubung langsung ke kedua jaringan antar-router, biasanya tidak perlu static route tambahan untuk jaringan 1.0 dan 3.0 jika sudah directly connected.

Namun jika sebelumnya dibuat static route tambahan, bisa muncul jalur ganda.

---

### ⚠ Troubleshooting Routing Ganda

<img width="596" height="462" alt="truble shoot 1" src="https://github.com/user-attachments/assets/6fc4b61e-4c81-4dd0-b138-a991f85b32b8" />


Terlihat bahwa jaringan 192.168.1.0 memiliki dua jalur karena sebelumnya dilakukan konfigurasi menggunakan metode berbeda.

Untuk menghapus static route yang tidak diperlukan:

`no ip route 192.168.1.0 255.255.255.0 192.168.12.1`

<img width="420" height="121" alt="trubleshoot 2" src="https://github.com/user-attachments/assets/d13903c4-0b67-413e-b938-6bdde1144731" />

Sekarang jalur yang tidak diperlukan sudah tidak muncul lagi.

Gunakan:

`show ip route`

untuk memastikan routing table sudah benar.

---

# 4️⃣ Pengujian Konektivitas

<img width="813" height="647" alt="ping ke jaringan 3" src="https://github.com/user-attachments/assets/34658442-9443-4f47-a831-a8791878c8d7" />

Lakukan pengujian dengan perintah:

`ping 192.168.3.1`


atau dari PC jaringan 1 menuju PC jaringan 3.

Pada percobaan pertama, paket mungkin belum terkirim karena proses ARP.

Pada percobaan kedua biasanya akan berhasil dan muncul:


---

# 📚 Konsep Penting Static Route

Format dasar:

ip route <destination-network> <subnet-mask> <next-hop-ip>

Static route bersifat:

- Manual
- Tidak otomatis berubah
- Cocok untuk jaringan kecil
- Lebih ringan dibanding dynamic routing

---

# 🧪 Troubleshooting Tambahan

Jika ping gagal, periksa:

- Apakah semua interface sudah `no shutdown`?
- Apakah IP address dan subnet mask benar?
- Apakah next-hop mengarah ke IP yang benar?
- Gunakan:
show ip interface brief
show ip route


---

# 💾 Menyimpan Konfigurasi

Setelah semua berhasil, simpan konfigurasi:

copy running-config startup-config, write, atau wr


---

✍️ Dokumentasi ini merupakan bagian dari perjalanan belajar Computer Networking dan praktik routing statis.








