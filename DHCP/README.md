# Panduan Konfigurasi DHCP (Dynamic Host Configuration Protocol) & DHCP Relay Agent

Dokumentasi ini berisi penjelasan konsep dasar DHCP, mekanisme empat langkah kerja (DORA), manfaat penting DHCP, serta praktik langsung (*hands-on*) konfigurasi DHCP Server, DHCP Client pada Interface Router, dan DHCP Relay Agent pada perangkat Cisco.

---

## 1. Konsep Dasar DHCP

**DHCP (Dynamic Host Configuration Protocol)** adalah protokol jaringan yang secara otomatis memberikan alamat IP dan konfigurasi lain (seperti subnet mask, default gateway, dan DNS) ke setiap perangkat yang terhubung. Tanpa DHCP, Anda harus mengonfigurasi pengaturan jaringan secara manual di setiap komputer, server, atau ponsel agar bisa saling terhubung.

###  Cara Kerja DHCP (4 Langkah Utama - DORA)
Proses otomatisasi ini terjadi di latar belakang dalam hitungan detik melalui empat tahapan utama:

1. **Discovery (DHCPDISCOVER):** Perangkat (*client*) mengirimkan paket *broadcast* untuk mencari server DHCP yang aktif di jaringan.
2. **Offer (DHCPOFFER):** Server DHCP merespons dengan menawarkan ketersediaan alamat IP beserta parameter konfigurasi jaringan kepada *client*.
3. **Request (DHCPREQUEST):** Perangkat (*client*) menyetujui penawaran tersebut dan meminta secara resmi penggunaan alamat IP yang ditawarkan.
4. **Acknowledgement (DHCPACK):** Server mengonfirmasi permintaan, memberikan alamat IP tersebut, dan mencatat batas waktu peminjamannya (*lease time*).

###  Mengapa DHCP Sangat Penting?
* **Otomatis & Praktis:** Tidak perlu memasukkan deretan angka IP secara manual saat menghubungkan perangkat baru ke WiFi atau LAN.
* **Mencegah Konflik IP:** Sistem secara otomatis memastikan setiap perangkat mendapatkan alamat IP yang unik.
* **Manajemen Jaringan Efisien:** Jika alamat IP habis masa sewanya (*lease expired*), server dapat mendaur ulang alamat tersebut untuk perangkat lain yang membutuhkan.

---

## 2. Skenario & Topologi Lab

<img width="668" height="382" alt="DHCP topologi" src="https://github.com/user-attachments/assets/8bfc6788-5bc1-4f40-bbab-be41934c9fa9" />

*(Gambar Topologi DHCP)*

Pada studi kasus kali ini, kita akan melakukan konfigurasi DHCP Server di **R2** untuk melayani beberapa subnet, mengonfigurasi interface **R1** sebagai DHCP Client, serta mengonfigurasi **R1** sebagai DHCP Relay Agent untuk meneruskan permintaan dari LAN ke R2.

---

### Langkah 1: Konfigurasi DHCP Server pada R2

#### a. Mengecualikan Alamat IP (Excluded Addresses)
Sebelum membuat *pool*, kita harus mengecualikan (*exclude*) alamat IP tertentu (seperti IP gateway, IP interface router, atau IP server statis) agar tidak dibagikan secara otomatis ke *client*.

```
R2(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.10
R2(config)# ip dhcp excluded-address 192.168.2.1 192.168.2.10
R2(config)# ip dhcp excluded-address 103.0.113.1
```
Sintaks Dasar:

Rentang IP: `Router(config)# ip dhcp excluded-address <alamat_awal> <alamat_akhir>`

Tunggal IP: `Router(config)# ip dhcp excluded-address <alamat_IP>`

#### b. Konfigurasi DHCP Pool (POOL1)
Cuplikan kode
```
R2(config)# ip dhcp pool POOL1
R2(dhcp-config)# network 192.168.1.0 255.255.255.0
R2(dhcp-config)# dns-server 8.8.8.8
R2(dhcp-config)# domain-name networklab.com
R2(dhcp-config)# default-router 192.168.1.1
```
Penjelasan kode di atas: 

| Perintah                            | Fungsi                                                                                                                                                                            | Contoh Hasil pada Client                                                         |
| ----------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| `ip dhcp pool POOL1`                | Membuat DHCP Pool (kumpulan konfigurasi DHCP) dengan nama **POOL1**. Nama ini hanya sebagai identitas dan bisa diganti sesuai kebutuhan, misalnya `KANTOR`, `LAB`, atau `VLAN10`. | Router membuat pool DHCP bernama **POOL1**.                                      |
| `network 192.168.1.0 255.255.255.0` | Menentukan jaringan yang akan dilayani DHCP. Semua IP yang dibagikan berasal dari subnet **192.168.1.0/24**, kecuali IP yang sudah di-*exclude*.                                  | Client akan mendapat IP, misalnya `192.168.1.11/24`.                             |
| `dns-server 8.8.8.8`                | Menentukan alamat DNS Server yang akan dikirim ke client. DNS digunakan untuk menerjemahkan nama domain (misalnya `google.com`) menjadi alamat IP.                                | Client otomatis menggunakan DNS `8.8.8.8`.                                       |
| `domain-name networklab.com`        | Memberikan nama domain lokal kepada client. Umumnya digunakan pada jaringan perusahaan atau laboratorium.                                                                         | Nama lengkap (FQDN) client bisa menjadi `PC1.networklab.com` jika dikonfigurasi. |
| `default-router 192.168.1.1`        | Menentukan Default Gateway yang digunakan client untuk berkomunikasi ke luar subnet atau ke internet. Biasanya alamat interface router pada jaringan tersebut.                    | Client menggunakan `192.168.1.1` sebagai gateway.                                |

####  c. Konfigurasi DHCP Pool (POOL2)

Cuplikan kode
```
R2(config)# ip dhcp pool POOL2
R2(dhcp-config)# network 192.168.2.0 255.255.255.0
R2(dhcp-config)# dns-server 8.8.8.8
R2(dhcp-config)# domain-name networklab.com
R2(dhcp-config)# default-router 192.168.2.1
```


#### d. Konfigurasi DHCP Pool (POOL3)
Cuplikan kode
```
R2(config)# ip dhcp pool POOL3
R2(dhcp-config)# network 203.0.113.0 255.255.255.252
```
Verifikasi Uji Coba Request DHCP pada PC2
Cobalah untuk mengubah mode IP Configuration dari Static ke DHCP pada PC2.


<img width="561" height="285" alt="request dhcp pc2" src="https://github.com/user-attachments/assets/e45f46db-c91f-4f5f-9c6f-b653c6ea6da4" />

(Gambar Request DHCP PC2)

Hasil: Request berhasil dan PC2 mendapatkan alamat IP address secara otomatis sesuai dengan alokasi pada POOL2.

### Langkah 2: Mengonfigurasi Interface g0/0 pada R1 sebagai DHCP Client
Router juga dapat bertindak sebagai DHCP Client untuk mendapatkan IP otomatis dari perangkat ISP atau router upstream.

```
R1(config)# interface g0/0
R1(config-if)# ip address dhcp
```
penjelasan kode:


| Perintah          | Fungsi                                                  | Penjelasan                                                                                                         |
| ----------------- | ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| `interface g0/0`  | Masuk ke mode konfigurasi interface GigabitEthernet0/0. | Semua konfigurasi berikutnya akan diterapkan pada interface G0/0.                                                  |
| `ip address dhcp` | Mengaktifkan DHCP Client pada interface.                | Router akan meminta IP Address, Subnet Mask, Gateway, dan informasi DHCP lainnya secara otomatis dari DHCP Server. |

### Langkah 3: Konfigurasi R1 sebagai DHCP Relay Agent (192.168.1.0/24)
Karena pesan DHCP Discovery menggunakan protokol broadcast (UDP port 67/68) yang secara default akan diblokir oleh router, maka R1 yang berada di antara client LAN dan DHCP Server (R2) perlu dikonfigurasi sebagai DHCP Relay Agent menggunakan perintah ip helper-address.

Cuplikan kode
```
R1(config)# interface g0/1
R1(config-if)# ip helper-address 203.0.113.1
```
penjelasan konfigurasi: 

| Perintah                        | Fungsi                                                     | Penjelasan                                                                                                             |
| ------------------------------- | ---------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `interface g0/1`                | Masuk ke mode konfigurasi interface GigabitEthernet0/1.    | Interface ini biasanya menghadap ke jaringan LAN tempat client berada.                                                 |
| `ip helper-address 203.0.113.1` | Mengaktifkan DHCP Relay dan menentukan alamat DHCP Server. | Router akan meneruskan permintaan DHCP dari client ke DHCP Server yang berada di jaringan lain, yaitu **203.0.113.1**. |

### Langkah 4: Uji Coba & Pengujian Akhir via CLI
1. Buka Command Prompt (CLI) pada PC1 dan PC2, lalu jalankan perintah perbarui IP:

   `C:\> ipconfig /renew`

2. Amati apakah PC1 mendapatkan IP dari subnet 192.168.1.0/24 via Relay Agent R1.

3. Tambahkan PC3 pada switch di bawah R1 dan PC4 pada switch di bawah R2 untuk memastikan seluruh sistem DHCP Pool dan Relay Agent dapat mengalokasikan alamat IP baru secara otomatis tanpa kendala.












