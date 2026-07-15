# Panduan Konfigurasi DNS (Domain Name System)

Dokumentasi ini berisi panduan konsep dasar, tabel penjelasan perintah, analisis konfigurasi, serta praktik langsung (*hands-on*) simulasi DNS Server pada router Cisco dan client di dalam jaringan.

---

## 1. Konsep Dasar DNS

**DNS (Domain Name System)** adalah sistem yang berfungsi menerjemahkan nama domain situs web (seperti `google.com`) menjadi alamat IP numerik (seperti `142.250.190.46`). [cite_start]Anda bisa menganggapnya sebagai "buku telepon internet" yang memudahkan manusia untuk mengingat alamat situs web tanpa harus menghafal kombinasi angka yang rumit.

Dalam modul ini, kita akan mencoba mengonfigurasi server DNS agar terhubung dengan situs web yang kita inginkan[cite: 3]. [cite_start]Pada contoh ini, kita akan mensimulasikan penggunaan DNS Server untuk menghubungkan jaringan kita ke YouTube menggunakan layanan DNS[cite: 4].

---

## 2. Skenario & Langkah-Langkah Konfigurasi

<img width="868" height="383" alt="topologi_dns" src="https://github.com/user-attachments/assets/18e7bf71-9225-44cc-9ac6-cb2d3704c695" />


Berikut adalah langkah-langkah konfigurasi yang harus kita lakukan:

### Langkah 1: Konfigurasi Default Route pada R1

Default route digunakan untuk melemparkan semua paket yang tidak diketahui tujuannya ke arah internet melalui IP gateway tetangga (*next hop*) di `203.0.113.2`.

```
R1(config)# interface g0/1
R1(config-if)# exit
R1(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.2
```

(Catatan perbaikan: Perintah routing ip route diketik pada global configuration mode, bukan di dalam sub-interface g0/1).

Setelah konfigurasi selesai, lakukan verifikasi koneksi dengan melakukan ping dari R1 ke alamat IP DNS publik (1.1.1.1) untuk memastikan default route sudah berhasil:

```
R1# ping 1.1.1.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds: [cite: 5]
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/0/0 ms [cite: 6]
```

koneksi sepenuhnya berhasil

---

## Langkah 2: Konfigurasi DNS Server di Semua PC

Pada setiap PC Client, buka menu IP Configuration dan isi kolom DNS Server dengan IP DNS publik yang kita gunakan, yaitu 1.1.1.1.

---

## Langkah 3: Konfigurasi R1 untuk Menggunakan DNS Server dan Entri Host Lokal

Kita akan mengonfigurasi R1 agar mengenali DNS Server 1.1.1.1, sekaligus mendaftarkan pemetaan IP Host secara lokal untuk jaringan di bawahnya.  

```
R1(config)# ip name-server 1.1.1.1
R1(config)# ip host R1 192.168.1.254
R1(config)# ip host PC1 192.168.1.1
R1(config)# ip host PC2 192.168.1.2
R1(config)# ip host PC3 192.168.1.3
```
📋 Penjelasan Perintah Konfigurasi

| Command / Action | Purpose |
|------------------|---------|
| `ip name-server 1.1.1.1` | Mengonfigurasi alamat IP DNS Server yang akan digunakan router untuk melakukan **DNS name resolution** (menerjemahkan nama domain menjadi alamat IP). |
| `ip host R1 192.168.1.254` | Membuat entri host lokal pada router yang memetakan nama **R1** ke alamat IP **192.168.1.254**. |
| `ip host PC1 192.168.1.1` | Membuat entri host lokal yang memetakan nama **PC1** ke alamat IP **192.168.1.1**. |
| `ip host PC2 192.168.1.2` | Membuat entri host lokal yang memetakan nama **PC2** ke alamat IP **192.168.1.2**.  |
| `ip host PC3 192.168.1.3` | Membuat entri host lokal yang memetakan nama **PC3** ke alamat IP **192.168.1.3**.  |

🔍 Verifikasi Tabel Host lokal
Kita dapat memeriksa apakah seluruh konfigurasi tabel host lokal sudah tersimpan dengan sukses menggunakan perintah berikut:

```
R1(config)# do show hosts
Default Domain is not set
Name/address lookup uses domain service
Name servers are 1.1.1.1

Codes: UN - unknown, EX - expired, OK - OK, ?? - revalidate [cite: 13]
       temp - temporary, perm - permanent
       NA - Not Applicable None - Not defined [cite: 13]

Host                      Port  Flags      Age Type   Address(es)
PC1                       None  (perm, OK)  0   IP      192.168.1.1 [cite: 13]
PC2                       None  (perm, OK)  0   IP      192.168.1.2 [cite: 14]
PC3                       None  (perm, OK)  0   IP      192.168.1.3 [cite: 14]
R1                        None  (perm, OK)  0   IP      192.168.1.254 [cite: 14, 15]
```
Dengan terdaftarnya nama-nama tersebut di database lokal router , kita bisa melakukan tes koneksi langsung ke PC client lain tanpa harus mengingat atau mengetik alamat IP-nya, cukup panggil nama host-nya saja.


🧪 Uji Coba Ping Berbasis Nama Host dari R1 ke PC

```
R1(config)# do ping PC1 [cite: 16]

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds: [cite: 17]
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 0/5/21 ms [cite: 18]
```
Ping berhasil dijalankan dengan lancar menggunakan translasi nama lokal.

## Langkah 4: Uji Coba Klien (Web Browser)
Untuk melakukan verifikasi akhir pada sisi client:

Buka aplikasi Web Browser di PC.

Ketik nama domain youtube.com pada kolom URL.

<img width="697" height="591" alt="uji dns" src="https://github.com/user-attachments/assets/4799092e-1d07-4154-80b1-1187cd913d30" />


Jika halaman berhasil terbuka, berarti konfigurasi DNS server kita telah sukses bekerja seutuhnya.

### 💡 Tugas Mandiri / Eksperimen
Sebagai latihan tambahan mandiri, silakan masuk ke Command Prompt salah satu PC (misalnya PC1), lalu lakukan ping ke PC2 dan PC3 menggunakan nama host-nya saja (misal: ping PC2). Amati dan analisis apa yang terjadi pada proses translasi tersebut!

