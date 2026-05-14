# Panduan Dasar IPv6: Pengertian dan Implementasi

Dokumentasi ini berisi catatan fundamental mengenai **IPv6 (Internet Protocol version 6)**, mulai dari alasan penggunaannya hingga aturan penulisan strukturnya.

---

## 1. Apa itu IPv6 dan Mengapa Kita Butuh?

IPv6 adalah protokol generasi terbaru yang dirancang untuk menggantikan IPv4 yang kapasitasnya mulai menipis. Berikut adalah alasan utama transisi ke IPv6:

*   **Kapasitas Tak Terbatas:** IPv6 menyediakan sekitar $3.4 \times 10^{38}$ alamat unik. Secara teori, jumlah ini cukup untuk memberikan alamat IP bagi setiap atom di permukaan bumi.
*   **Efisiensi Routing:** Header paket IPv6 dibuat lebih sederhana dibandingkan IPv4, sehingga beban kerja router berkurang dan pemrosesan data menjadi lebih cepat.
*   **Keamanan Bawaan:** Protokol **IPsec** (Internet Protocol Security) sudah terintegrasi secara *native* dalam desain IPv6 sejak awal, memberikan enkripsi dan autentikasi yang lebih baik.

---

## 2. Format dan Anatomi Alamat

Berbeda dengan IPv4 yang menggunakan angka desimal dan pemisah titik (contoh: `192.168.1.1`), IPv6 menggunakan format **heksadesimal** dan pemisah **titik dua (colon)**.

### Struktur Dasar:
*   Terdiri dari **128 bit**.
*   Dibagi menjadi **8 blok**, di mana masing-masing blok berisi **4 digit heksadesimal** (16 bit per blok).

**Contoh Alamat Lengkap:**
`2001:0db8:85a3:0000:0000:8a2e:0370:7334`

---

## 3. Aturan Penyederhanaan

Agar penulisan tidak terlalu panjang, terdapat dua aturan resmi untuk menyingkat alamat IPv6:

### A. Omit Leading Zeros (Menghapus Nol di Depan)
Angka nol di posisi awal setiap blok boleh dihapus.
*   `0db8` menjadi `db8`
*   `0000` menjadi `0`

### B. Double Colon (::)
Blok berurutan yang berisi angka nol semua dapat diganti dengan tanda titik dua ganda (`::`).
> **Penting:** Aturan ini hanya boleh dilakukan **satu kali** dalam satu alamat untuk menghindari ambiguitas.

#### Contoh Transformasi:
| Tahap | Format Alamat |
| :--- | :--- |
| **Alamat Asli** | `2001:0db8:0000:0000:0000:0000:0000:0001` |
| **Sederhana (Tanpa Nol Depan)** | `2001:db8:0:0:0:0:0:1` |
| **Paling Sederhana (::)** | `2001:db8::1` |

---

## Ringkasan Perbandingan
| Fitur | IPv4 | IPv6 |
| :--- | :--- | :--- |
| Panjang Alamat | 32-bit | 128-bit |
| Format | Desimal | Heksadesimal |
| Pemisah | Titik (`.`) | Titik Dua (`:`) |
| Jumlah Alamat | ~4,3 Miliar | ~340 Undesiliun |

## 4. Contoh Implementasi Topologi

<img width="411" height="337" alt="IPv6" src="https://github.com/user-attachments/assets/d2fd9b7c-550f-482e-8bb9-a7645e563f7c" />


Pada bagian ini, kita akan mengonfigurasi **Dual-Stack** (menjalankan IPv4 dan IPv6 secara bersamaan) pada satu router (R1) yang menghubungkan tiga jaringan berbeda.

### Addressing Table

| Device | Interface | IPv4 Address | IPv6 Address | Prefix | Default Gateway |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **R1** | G0/0 | 192.168.1.1 | 2001:db8:0:1::1 | /24 (v4) /64 (v6) | N/A |
| | G0/1 | 192.168.2.1 | 2001:db8:0:2::1 | /24 (v4) /64 (v6) | N/A |
| | G0/2 | 192.168.3.1 | 2001:db8:0:3::1 | /24 (v4) /64 (v6) | N/A |
| **PC1** | NIC | 192.168.1.2 | 2001:db8:0:1::2 | /24 (v4) /64 (v6) | 192.168.1.1 / fe80::1 |
| **PC2** | NIC | 192.168.2.2 | 2001:db8:0:2::2 | /24 (v4) /64 (v6) | 192.168.2.1 / fe80::1 |
| **PC3** | NIC | 192.168.3.2 | 2001:db8:0:3::2 | /24 (v4) /64 (v6) | 192.168.3.1 / fe80::1 |

---

## 5. Langkah Konfigurasi & Troubleshooting

Bagian ini menjelaskan proses konfigurasi *Dual-Stack* pada Router R1 serta solusi untuk masalah konektivitas yang umum ditemui.

Berikut adalah perintah untuk mengaktifkan IPv6 dan mengatur alamat pada interface Router R1:


### A. Konfigurasi pada Router R1

Masuk ke mode konfigurasi interface dan masukkan alamat sesuai dengan *addressing table*:

#### Konfigurasi IPv6
```
R1(config)# interface g0/0
R1(config-if)# ipv6 address 2001:db8:0:1::1/64
R1(config-if)# interface g0/1
R1(config-if)# ipv6 address 2001:db8:0:2::1/64
R1(config-if)# interface g0/2
R1(config-if)# ipv6 address 2001:db8:0:3::1/64
```

#### Konfigurasi IPv4
```
R1(config-if)# interface g0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# interface g0/1
R1(config-if)# ip address 192.168.2.1 255.255.255.0
R1(config-if)# interface g0/2
R1(config-if)# ip address 192.168.3.1 255.255.255.0\
```

### B. Konfigurasi End Devices & Troubleshooting Penting
Setelah mengonfigurasi IP pada setiap PC sesuai tabel, mungkin kamu akan menemui masalah saat mencoba ping antar jaringan (misal dari PC1 ke PC2) menggunakan IPv6, di mana hasilnya adalah Request Timed Out (RTO).

Mengapa RTO Terjadi?
Meskipun alamat IP sudah terpasang, router tidak akan meneruskan paket IPv6 antar interface secara otomatis. Secara default, router bertindak seperti Host biasa sampai kita mengaktifkan fungsi routingnya.

<img width="414" height="415" alt="IPv6 error" src="https://github.com/user-attachments/assets/65f06865-32dd-4b72-8784-7087fe69ad7a" />

Solusi: Tombol "ON" Router IPv6
Gunakan perintah berikut di mode konfigurasi global:

`R1(config)# ipv6 unicast-routing`

#### Analogi:
ipv6 unicast-routing adalah tombol "ON" untuk fungsi routing pada protokol IPv6. Tanpa perintah ini, router tidak akan tahu cara mengirimkan paket dari satu segmen jaringan ke segmen lainnya.

Hasil Setelah Perbaikan:
Setelah perintah diaktifkan, hasil ping akan menunjukkan Reply, menandakan komunikasi antar VLAN/jaringan sudah berhasil.

Kesimpulan
Konfigurasi IPv6 sebenarnya memiliki kemiripan logika dengan IPv4. Jika kamu sudah memahami dasar-dasar IPv4 (seperti pengalamatan dan gateway), transisi ke IPv6 akan terasa lebih mudah. Poin utamanya adalah membiasakan diri dengan format heksadesimal dan jangan lupa mengaktifkan fitur routing IPv6 di perangkat pusat.



