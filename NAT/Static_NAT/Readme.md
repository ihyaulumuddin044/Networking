# NAT (Network Address Translation)

**NAT (Network Address Translation)** adalah teknologi jaringan komputer yang bertugas menerjemahkan alamat IP dari satu ruang alamat ke ruang alamat lainnya. Dalam penggunaan jaringan lokal, NAT umumnya digunakan untuk menerjemahkan alamat IP private pada perangkat internal menjadi alamat IP global/public ketika berkomunikasi dengan jaringan luar atau Internet.

## Fungsi Utama NAT

* Mengubah alamat IP private menjadi alamat IP global/public dan sebaliknya.
* Memungkinkan banyak komputer lokal berbagi alamat IP global/public, khususnya menggunakan PAT/Overload.
* Menyembunyikan alamat IP internal dari jaringan luar.
* Menghemat penggunaan alamat IP public yang terbatas.

## Jenis-Jenis NAT

* **NAT Statis:** Menerjemahkan satu IP private ke satu IP global secara tetap.
* **NAT Dinamis:** Menerjemahkan IP private ke IP global dari sekumpulan alamat yang tersedia secara dinamis.
* **PAT / Overload:** Memungkinkan banyak IP private menggunakan satu IP global dengan membedakan setiap koneksi menggunakan nomor port.

---

# NAT Statis

**NAT Statis (Static Network Address Translation)** adalah metode penerjemahan alamat IP pada router yang memetakan satu alamat IP lokal secara tetap ke satu alamat IP global.

Berbeda dengan NAT dinamis dan PAT, pemetaan pada Static NAT bersifat **one-to-one** dan tidak berubah selama konfigurasi NAT tersebut masih aktif.

Pada studi kasus ini, kita akan melakukan konfigurasi **Static NAT** pada R1 untuk memetakan PC1, PC2, dan PC3 ke alamat global masing-masing.

<img width="908" height="431" alt="Screenshot 2026-08-22 173057" src="https://github.com/user-attachments/assets/a8d5ac15-23df-4a04-81e2-64ab3fde8ebe" />

*(Gambar Topologi NAT)*

---

# 1. Menguji Koneksi PC1 ke Internet

Sebelum melakukan konfigurasi NAT, kita akan mencoba melakukan ping dari PC1 menuju `8.8.8.8`.

```text id="9s2qhv"
C:\>ping 8.8.8.8

Pinging 8.8.8.8 with 32 bytes of data:

Reply from 172.16.0.254: Destination host unreachable.
Reply from 172.16.0.254: Destination host unreachable.
Reply from 172.16.0.254: Destination host unreachable.
Reply from 172.16.0.254: Destination host unreachable.

Ping statistics for 8.8.8.8:
    Packets: Sent = 4, Received = 0, Lost = 4 (100% loss),
```

Hasilnya adalah:

```text
Destination host unreachable
```

Artinya, PC1 belum berhasil mencapai tujuan `8.8.8.8`.

Pada tahap ini NAT juga belum dikonfigurasi, sehingga alamat sumber dari jaringan internal masih menggunakan alamat lokal:

```text
172.16.0.1
```

Agar komunikasi menuju jaringan luar dapat berjalan, nantinya kita membutuhkan konfigurasi NAT serta routing yang sesuai.

---

# 2. Konfigurasi NAT Statis pada R1

Langkah pertama adalah menentukan interface mana yang mengarah ke jaringan internal (**NAT Inside**) dan interface mana yang mengarah ke jaringan luar (**NAT Outside**).

Konfigurasikan:

```text id="7r70w6"
R1(config)#interface g0/0
R1(config-if)#ip nat outside
R1(config-if)#interface g0/1
R1(config-if)#ip nat inside
```

### Penjelasan

| Command          | Arti                                  | Fungsi                                                                      |
| ---------------- | ------------------------------------- | --------------------------------------------------------------------------- |
| `interface g0/0` | Masuk ke interface GigabitEthernet0/0 | Memilih interface yang akan dikonfigurasi.                                  |
| `ip nat outside` | Menandai G0/0 sebagai **NAT Outside** | Memberi tahu router bahwa interface ini mengarah ke jaringan luar/Internet. |
| `interface g0/1` | Masuk ke interface GigabitEthernet0/1 | Memilih interface berikutnya.                                               |
| `ip nat inside`  | Menandai G0/1 sebagai **NAT Inside**  | Memberi tahu router bahwa interface ini mengarah ke jaringan internal/LAN.  |

Secara sederhana:

```text
LAN / Private Network
       │
       │
    G0/1
   NAT INSIDE
      R1
   NAT ROUTER
      R1
   NAT OUTSIDE
    G0/0
       │
       │
  ISP / Internet
```

Dengan konfigurasi tersebut, R1 sudah mengetahui batas antara jaringan **inside** dan **outside** untuk proses NAT.

---

## Membuat Pemetaan Static NAT

Selanjutnya kita akan membuat pemetaan antara IP PC1, PC2, dan PC3 dengan alamat global masing-masing.

Pada lab ini digunakan pemetaan:

| Perangkat | Inside Local | Inside Global |
| --------- | ------------ | ------------- |
| PC1       | `172.16.0.1` | `100.0.0.1`   |
| PC2       | `172.16.0.2` | `100.0.0.2`   |
| PC3       | `172.16.0.3` | `100.0.0.3`   |

Konfigurasikan:

```text id="cv9k4k"
R1(config)#ip nat inside source static 172.16.0.1 100.0.0.1
R1(config)#ip nat inside source static 172.16.0.2 100.0.0.2
R1(config)#ip nat inside source static 172.16.0.3 100.0.0.3
R1(config)#exit
R1#
%SYS-5-CONFIG_I: Configured from console by console
```

Kemudian periksa tabel translasi NAT:

```text id="55ps9t"
R1#show ip nat translations

Pro  Inside global     Inside local       Outside local      Outside global
---  100.0.0.1         172.16.0.1         ---                ---
---  100.0.0.2         172.16.0.2         ---                ---
---  100.0.0.3         172.16.0.3         ---                ---
```

### Penjelasan

| Command                                            | Penjelasan                                                                                 |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| `ip nat inside source static 172.16.0.1 100.0.0.1` | Membuat **Static NAT** yang memetakan IP lokal `172.16.0.1` menjadi IP global `100.0.0.1`. |
| `ip nat inside source static 172.16.0.2 100.0.0.2` | Memetakan IP lokal `172.16.0.2` menjadi IP global `100.0.0.2`.                             |
| `ip nat inside source static 172.16.0.3 100.0.0.3` | Memetakan IP lokal `172.16.0.3` menjadi IP global `100.0.0.3`.                             |
| `exit`                                             | Keluar dari configuration mode.                                                            |
| `show ip nat translations`                         | Menampilkan tabel translasi NAT yang dibuat oleh router.                                   |

---

## Memahami Tabel NAT

Perhatikan hasil:

```text id="7n3tdq"
Inside global     Inside local
100.0.0.1         172.16.0.1
100.0.0.2         172.16.0.2
100.0.0.3         172.16.0.3
```

Dua istilah yang sangat penting:

### Inside Local

**Inside Local** adalah alamat asli perangkat di dalam jaringan internal.

Contohnya:

```text id="9m3f3j"
172.16.0.1
```

Ini adalah alamat IP asli PC1 pada jaringan lokal.

### Inside Global

**Inside Global** adalah alamat yang digunakan untuk merepresentasikan perangkat internal ketika berkomunikasi melalui sisi luar router.

Contohnya:

```text id="y5x2mg"
100.0.0.1
```

Sehingga secara sederhana:

```text id="u5c5ms"
PC1
172.16.0.1
     │
     │ NAT
     ▼
R1
     │
     ▼
100.0.0.1
```

Dengan demikian, ketika PC1 mengirimkan traffic menuju jaringan luar, R1 dapat menerjemahkan alamat sumber `172.16.0.1` menjadi `100.0.0.1`.

> **Catatan tentang alamat `100.0.0.0/24`**
>
> Pada lab ini `100.0.0.0/24` digunakan sebagai contoh alamat **global** untuk mensimulasikan hasil translasi NAT. Dalam jaringan nyata, alamat global/public harus berasal dari ruang alamat yang memang dialokasikan untuk penggunaan tersebut. Untuk dokumentasi atau simulasi jaringan, sebaiknya gunakan juga rentang dokumentasi seperti `203.0.113.0/24` apabila ingin menghindari penggunaan alamat publik yang sebenarnya.

---

# 3. Menguji Konfigurasi NAT

Setelah Static NAT berhasil dibuat, kita dapat melakukan pengujian untuk memastikan bahwa proses translasi berjalan sesuai dengan konfigurasi.

<img width="847" height="605" alt="Screenshot 2026-08-22 173049" src="https://github.com/user-attachments/assets/8687d6ba-6e94-44e5-8514-88f98554e8d4" />

*(Gambar Konfigurasi NAT)*

Lakukan pengujian konektivitas dari PC1, PC2, dan PC3 menuju jaringan luar sesuai dengan topologi yang telah dibuat.

Contohnya:

```text
PC1 → R1 → ISP
PC2 → R1 → ISP
PC3 → R1 → ISP
```

Kemudian periksa kembali tabel translasi pada R1 menggunakan:

```text
R1#show ip nat translations
```

Perhatikan apakah pemetaan berikut tetap tersedia:

```text
172.16.0.1 → 100.0.0.1
172.16.0.2 → 100.0.0.2
172.16.0.3 → 100.0.0.3
```

Jika traffic dari jaringan internal sudah melewati R1, tabel NAT dapat digunakan untuk melihat bagaimana router melakukan translasi alamat.

> **Catatan penting**
>
> Static NAT hanya bertugas melakukan **translation**. NAT tidak menggantikan fungsi routing.
>
> Jadi jika PC masih belum dapat mencapai `8.8.8.8` setelah Static NAT dibuat, jangan langsung menyimpulkan bahwa konfigurasi NAT salah. Periksa juga:
>
> * Default gateway PC.
> * Routing pada R1.
> * Default route R1 menuju ISP.
> * Routing balik pada router ISP.
> * Konfigurasi `ip nat inside` dan `ip nat outside`.
> * Konektivitas antar-interface.
>
> Dengan kata lain:
>
> **Routing menentukan ke mana paket pergi, sedangkan NAT menentukan bagaimana alamat IP paket tersebut diterjemahkan.**

---

# Kesimpulan

Pada studi kasus ini kita telah mempelajari konsep dasar **Static NAT** dan cara melakukan konfigurasi pada router Cisco.

Konsep utamanya adalah:

```text
Inside Local  →  Inside Global
172.16.0.1    →  100.0.0.1
172.16.0.2    →  100.0.0.2
172.16.0.3    →  100.0.0.3
```

Static NAT menggunakan pemetaan **one-to-one**, sehingga setiap alamat IP internal memiliki pasangan alamat global yang tetap.

Kita juga mempelajari bahwa konfigurasi NAT membutuhkan penentuan sisi:

```text
ip nat inside
ip nat outside
```

serta pembuatan aturan translasi:

```text
ip nat inside source static <inside-local> <inside-global>
```

Terakhir, kita dapat memeriksa hasil konfigurasi menggunakan:

```text
show ip nat translations
```

Perlu diingat bahwa **NAT bukan routing**. Agar komunikasi ke Internet benar-benar berhasil, NAT harus didukung oleh konfigurasi routing yang benar pada seluruh jalur komunikasi.
