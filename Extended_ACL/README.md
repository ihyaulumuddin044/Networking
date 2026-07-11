# Panduan Konfigurasi Extended ACLs (Access Control Lists)

Dokumentasi ini berisi panduan komprehensif mengenai konsep dasar, perbedaan dengan Standard ACL, serta praktik langsung (*hands-on*) penerapan Extended ACL menggunakan *Numbered* maupun *Named* ACL pada perangkat Cisco Jaringan untuk memblokir layanan spesifik (DNS, HTTP, dan HTTPS).

---

## 1. Konsep Dasar & Perbandingan ACL

**Extended ACL (Access Control List)** pada Cisco adalah fitur keamanan jaringan yang menyaring lalu lintas data secara spesifik berdasarkan alamat asal (*source*), tujuan (*destination*), jenis protokol, dan nomor port. Perbedaan utamanya dengan Standard ACL terletak pada tingkat ketelitian (granularitas) penyaringan dan lokasi penempatannya di dalam jaringan.

### 📋 Tabel Perbandingan: Standard ACL vs Extended ACL

| Fitur | Standard ACL | Extended ACL |
| :--- | :--- | :--- |
| **Kriteria Penyaringan** | Hanya memeriksa alamat IP asal (*source IP*). | Memeriksa IP asal, IP tujuan, jenis protokol (IP, TCP, UDP, ICMP), dan nomor port (seperti HTTP port 80, FTP port 21). |
| **Nomor ID (Numbered)** | `1–99` dan `1300–1999` | `100–199` dan `2000–2699` |
| **Lokasi Penempatan** | Harus diletakkan **sedekat mungkin dengan tujuan (*destination*)** agar tidak salah memblokir lalu lintas lain di tengah jalan. | Harus diletakkan **sedekat mungkin dengan sumber (*source*)** untuk menghemat *bandwidth* jaringan dengan membuang paket sebelum masuk lebih jauh. |
| **Analogi Terapan** | Seperti satpam yang menolak tamu hanya berdasarkan kota asal sang tamu, tanpa peduli tujuan atau keperluannya. | Seperti satpam yang memeriksa KTP tamu, melihat siapa yang ingin ditemui, dan menanyakan keperluan surat dinasnya secara spesifik. |

---

## 2. Skenario Uji Coba (Lab Extended ACL)

Pada implementasi kali ini, kita akan mengembangkan kebijakan keamanan jaringan menggunakan Extended ACL dengan dua aturan spesifik:
1. Host **172.16.1.1** tidak dapat mengakses layanan **DNS** di **Server 1** (`192.168.1.100`).
2. Host **172.16.2.1** tidak dapat mengakses layanan **HTTP** dan **HTTPS** di **Server 1** (`192.168.1.100`).

<img width="879" height="362" alt="standard_acl_topologi" src="https://github.com/user-attachments/assets/ef4ec848-380e-480c-81b3-b4b3adffe215" />

---

### Kasus 1: Blokir Layanan DNS dari Host 172.16.1.1 ke Server 1

Sebelum ACL diterapkan, host `172.16.1.1` dipastikan sukses melakukan *query* DNS ke Server 1 (dibuktikan dengan kemampuan melakukan ping menggunakan nama domain/hostname seperti `server1` atau `pc2` yang terdaftar pada DNS).

<img width="644" height="255" alt="gambar akses dns server1" src="https://github.com/user-attachments/assets/5983a572-43be-4ac2-8c7c-c489b77cdb0b" />

#### 🛠️ Konfigurasi Extended ACL pada R1
```
R1(config)# ip access-list extended 100
R1(config-ext-nacl)# deny udp 172.16.1.1 0.0.0.0 host 192.168.1.100 eq 53
R1(config-ext-nacl)# deny tcp 172.16.1.1 0.0.0.0 host 192.168.1.100 eq 53
R1(config-ext-nacl)# permit ip any any
```

### Analisis Struktur Perintah:

* ip access-list extended 100: Membuat atau masuk ke mode konfigurasi Extended ACL dengan nomor pengenal 100.

* deny: Aksi untuk menolak/memblokir paket data yang cocok dengan kriteria.

* udp / tcp: Menentukan protokol transport. Layanan DNS menggunakan UDP port 53 untuk query standar, dan TCP port 53 untuk transfer zona data besar. Kita memblokir keduanya demi keamanan.

* 172.16.1.1 0.0.0.0: IP asal (Source IP). Wildcard mask 0.0.0.0 mewakili kecocokan mutlak satu host (sama artinya dengan menuliskan kata kunci host 172.16.1.1).

* host 192.168.1.100: IP tujuan (Destination IP) yang mengarah spesifik ke Server 1.

* eq 53: singkatan dari Equal to 53. Port 53 adalah port standar global untuk layanan DNS.

* permit ip any any: Mengizinkan sisa lalu lintas IP lainnya dari sumber mana pun ke tujuan mana pun (implicit deny bypass).

📌 Penerapan pada Interface Terdekat (Inbound Sumber)
Sesuai aturan dasar Extended ACL, konfigurasi dipasang pada interface router terdekat dari PC pengirim (dalam kasus ini interface g0/1 pada R1) arah masuk (inbound).

```
R1(config)# interface g0/1
R1(config-if)# ip access-group 100 in
```
🔍 Hasil Uji Coba Kasus 1
Uji Coba Layanan DNS (Nama Domain):

```
C:\> ping pc2
Ping request could not find host pc2. Please check the name and try again.
Hasil menunjukkan error gagal menemukan host karena query nama domain ke DNS server telah berhasil diblokir.
```

Uji Coba Koneksi Dasar (Menggunakan IP Langsung):

```
C:\> ping 172.16.1.2

Pinging 172.16.1.2 with 32 bytes of data:
Reply from 172.16.1.2: bytes=32 time=1ms TTL=128
Reply from 172.16.1.2: bytes=32 time<1ms TTL=128

Ping statistics for 172.16.1.2:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
```
PC masih bisa melakukan koneksi ping langsung menggunakan alamat IP. Ini membuktikan Extended ACL bekerja secara granular: hanya memblokir fungsi DNS (port 53) tanpa mematikan koneksi dasar jaringan (ICMP/IP).

---

### Kasus 2: Blokir Layanan HTTP & HTTPS dari Host 172.16.2.1 ke Server 1
Sebelum aturan ini dipasang, PC3 (172.16.2.1) dapat membuka halaman website Server 1 secara normal melalui Web Browser Cisco Packet Tracer.

🛠️ Konfigurasi Extended ACL pada R1
Cuplikan kode
```
R1(config)# ip access-list extended 110
R1(config-ext-nacl)# deny tcp 172.16.2.1 0.0.0.0 host 192.168.1.100 eq 80
R1(config-ext-nacl)# deny tcp 172.16.2.1 0.0.0.0 host 192.168.1.100 eq 443
R1(config-ext-nacl)# permit ip any any
```
Analisis Struktur Perintah:

eq 80: Memblokir lalu lintas port 80 yang digunakan oleh protokol web tidak aman (HTTP).

eq 443: Memblokir lalu lintas port 443 yang digunakan oleh protokol web aman (HTTPS).

Kedua aturan di atas menggunakan protokol transport tcp karena layanan web mengandalkan koneksi TCP three-way handshake.

📌 Penerapan pada Interface Terdekat (Inbound Sumber)
Terapkan aturan ini pada interface yang menjadi gerbang masuk bagi host 172.16.2.1, yaitu interface g0/2 pada R1.

Cuplikan kode
```
R1(config)# interface g0/2
R1(config-if)# ip access-group 110 in
```
🔍 Hasil Uji Coba Kasus 2
Lakukan pengujian dengan membuka Web Browser pada PC3, kemudian ketik alamat IP Server 1 (192.168.1.100).

Sebelum Blokir: Halaman web terbuka menampilkan respons dari server.

<img width="666" height="587" alt="gambar sebelum blokir http" src="https://github.com/user-attachments/assets/72845bc2-a787-4082-927e-57d488a192d7" />


Setelah Blokir: Proses muat halaman akan mengalami kegagalan/berhenti (Timeout / Connection Refused), menandakan paket request HTTP (port 80) dan HTTPS (port 443) telah dijatuhkan secara sepihak oleh Router 1 sebelum sempat diteruskan ke luar segmen LAN.


<img width="679" height="609" alt="setelah blokir http" src="https://github.com/user-attachments/assets/539d663f-4250-4c65-bcbe-1c2d53dc24bc" />


💡 Kesimpulan Akhir
Melalui implementasi di atas, kita dapat menyimpulkan bahwa Extended ACL memberikan kendali penuh yang jauh lebih aman dan efisien jika dibandingkan dengan Standard ACL. Keuntungan utamanya meliputi kemampuan membatasi aplikasi tertentu (seperti memblokir web atau DNS saja) serta efisiensi beban router karena penyaringan dilakukan tepat di pintu masuk jaringan asal (source).













