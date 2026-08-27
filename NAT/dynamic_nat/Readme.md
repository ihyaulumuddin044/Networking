# Dynamic NAT

Dynamic NAT (Network Address Translation Dinamis) adalah metode penerjemahan alamat IP privat (lokal) menjadi alamat IP publik secara otomatis menggunakan kumpulan atau *pool* alamat IP publik yang tersedia.

## Cara Kerja Dynamic NAT

- Perangkat internal dengan IP privat ingin mengakses internet.
- Router mengambil salah satu IP publik yang kosong dari *pool* (kumpulan) yang sudah didaftarkan.
- Router memetakan IP privat tersebut ke IP publik secara **satu-ke-satu (one-to-one)** untuk sesi komunikasi tersebut.
- Saat koneksi selesai, IP publik akan dikembalikan ke *pool* agar bisa dipakai perangkat lain.

## Kelebihan dan Kekurangan

**Kelebihan:**
- Lebih fleksibel dibanding Static NAT karena pemetaan IP tidak permanen.
- IP publik dikelola secara otomatis oleh router.
- Satu *pool* IP publik dapat digunakan oleh beberapa perangkat secara bergantian.

**Kekurangan:**
- Jika jumlah IP publik di dalam *pool* habis, perangkat lain dalam jaringan lokal tidak bisa melakukan NAT sampai ada IP yang tersedia kembali.
- Tidak cocok jika banyak perangkat harus dapat mengakses internet secara bersamaan dengan jumlah IP publik yang terbatas.

---

# Studi Kasus

<img width="747" height="353" alt="dynamic nat" src="https://github.com/user-attachments/assets/76f64e61-57b0-443e-a2bc-e72553940ac6" />

(gambar topologi)

Pada studi kasus ini, kita akan membuat Dynamic NAT pada R1 menggunakan **ACL** dan **NAT Pool**.

## 1. Konfigurasi Dynamic NAT pada R1

### A. Konfigurasi Interface Inside dan Outside

```text
R1(config)#int g0/0
R1(config-if)#ip nat outside
R1(config-if)#int g0/1
R1(config-if)#ip nat inside
```
Untuk memahami perintah ini, kalian bisa mempelajari Static NAT pada materi sebelumnya.

| Command          | Arti                                  | Fungsi                                                           |
| ---------------- | ------------------------------------- | ---------------------------------------------------------------- |
| `int g0/0`       | Masuk ke interface GigabitEthernet0/0 | Memilih interface yang mengarah ke jaringan luar/Internet        |
| `ip nat outside` | Menandai G0/0 sebagai NAT Outside     | Memberi tahu router bahwa interface ini berada di sisi luar NAT  |
| `int g0/1`       | Masuk ke interface GigabitEthernet0/1 | Memilih interface yang mengarah ke jaringan lokal                |
| `ip nat inside`  | Menandai G0/1 sebagai NAT Inside      | Memberi tahu router bahwa interface ini berada di sisi dalam NAT |

### B. Membuat ACL dan NAT Pool

Kita akan menerjemahkan traffic yang berasal dari jaringan 192.168.0.0/24.

```
R1(config)#access-list 1 permit 192.168.0.0 0.0.0.255
R1(config)#ip nat pool POOL1 100.0.0.1 100.0.0.2 netmask 255.255.255.0
R1(config)#ip nat inside source list 1 pool POOL1
```
| Command                                                       | Bagian                  | Arti                                               |
| ------------------------------------------------------------- | ----------------------- | -------------------------------------------------- |
| `access-list 1 permit 192.168.0.0 0.0.0.255`                  | `access-list 1`         | Membuat Standard ACL nomor 1                       |
|                                                               | `permit`                | Mengizinkan traffic yang cocok dengan aturan       |
|                                                               | `192.168.0.0`           | Network sumber yang akan dikenai NAT               |
|                                                               | `0.0.0.255`             | Wildcard mask untuk `192.168.0.0/24`               |
| `ip nat pool POOL1 100.0.0.1 100.0.0.2 netmask 255.255.255.0` | `ip nat pool`           | Membuat NAT Pool                                   |
|                                                               | `POOL1`                 | Nama NAT Pool                                      |
|                                                               | `100.0.0.1`             | IP pertama dalam pool                              |
|                                                               | `100.0.0.2`             | IP terakhir dalam pool                             |
|                                                               | `netmask 255.255.255.0` | Subnet mask untuk alamat dalam pool                |
| `ip nat inside source list 1 pool POOL1`                      | `inside source`         | Melakukan NAT terhadap source dari jaringan inside |
|                                                               | `list 1`                | Menggunakan ACL nomor 1 sebagai penentu source     |
|                                                               | `pool POOL1`            | Menggunakan alamat IP dari NAT Pool `POOL1`        |

Gambaran Sederhana
Dengan konfigurasi tersebut, router memiliki:

```
LAN
192.168.0.0/24
     |
     |  NAT
     v
+----------------+
|      R1        |
+----------------+
     |
     v
NAT Pool
100.0.0.1
100.0.0.2
```
Misalnya PC1 192.168.0.1 ingin mengakses Internet, router dapat memberikan 100.0.0.1.

Jika PC2 192.168.0.2 juga mengakses Internet, router dapat memberikan 100.0.0.2.

Karena hanya tersedia 2 IP publik, maka pada saat kedua IP sedang digunakan, perangkat ketiga tidak mendapatkan IP dari pool.

---

## 2. Uji Coba Dynamic NAT

Sekarang kita coba melakukan ping dari PC1 ke google.com.

```
Cisco Packet Tracer PC1 Command Line 1.0
C:\>ping google.com

Pinging 8.8.8.8 with 32 bytes of data:

Reply from 8.8.8.8: bytes=32 time=11ms TTL=126
Reply from 8.8.8.8: bytes=32 time<1ms TTL=126
Reply from 8.8.8.8: bytes=32 time<1ms TTL=126
Reply from 8.8.8.8: bytes=32 time<1ms TTL=126

Ping statistics for 8.8.8.8:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 11ms, Average = 2ms
```
Ping berhasil. Dengan ini berarti PC1 sudah dapat terhubung ke server 8.8.8.8.

Selanjutnya kita periksa tabel translasi NAT pada R1:

```
R1#sh ip nat translations

Pro  Inside global     Inside local       Outside local      Outside global
icmp 100.0.0.1:29      192.168.0.1:29     8.8.8.8:29         8.8.8.8:29
icmp 100.0.0.1:30      192.168.0.1:30     8.8.8.8:30         8.8.8.8:30
icmp 100.0.0.1:31      192.168.0.1:31     8.8.8.8:31         8.8.8.8:31
icmp 100.0.0.1:32      192.168.0.1:32     8.8.8.8:32         8.8.8.8:32
icmp 100.0.0.2:1       192.168.0.2:1      8.8.8.8:1          8.8.8.8:1
icmp 100.0.0.2:2       192.168.0.2:2      8.8.8.8:2          8.8.8.8:2
icmp 100.0.0.2:3       192.168.0.2:3      8.8.8.8:3          8.8.8.8:3
icmp 100.0.0.2:4       192.168.0.2:4      8.8.8.8:4          8.8.8.8:4
udp 100.0.0.1:1026     192.168.0.1:1026   8.8.8.8:53         8.8.8.8:53
udp 100.0.0.2:1025     192.168.0.2:1025   8.8.8.8:53         8.8.8.8:53
```
Dari hasil tersebut terlihat bahwa:

```
192.168.0.1  →  100.0.0.1
192.168.0.2  →  100.0.0.2
```

Artinya Dynamic NAT berhasil melakukan pemetaan IP privat ke IP yang tersedia pada NAT Pool.

Perhatikan juga bahwa Inside Global berbeda dengan Inside Local:

| Istilah        | Contoh        | Keterangan                                                     |
| -------------- | ------------- | -------------------------------------------------------------- |
| Inside Local   | `192.168.0.1` | IP asli perangkat di jaringan internal                         |
| Inside Global  | `100.0.0.1`   | IP yang digunakan perangkat tersebut setelah diterjemahkan NAT |
| Outside Local  | `8.8.8.8`     | Alamat server luar sebagaimana terlihat dari jaringan internal |
| Outside Global | `8.8.8.8`     | Alamat asli server di jaringan luar                            |

Dengan demikian, konfigurasi Dynamic NAT kita sudah berjalan dengan baik.

---

## 3. Mengubah Dynamic NAT Menjadi PAT

Sekarang kita akan mengubah konfigurasi Dynamic NAT menjadi PAT (Port Address Translation) atau NAT Overload.

PAT memungkinkan banyak IP privat menggunakan satu IP publik dengan membedakan setiap koneksi menggunakan nomor port.

Pertama kita melihat konfigurasi NAT yang sedang berjalan:

```
R1#sh run | include nat

 ip nat outside
 ip nat inside
ip nat pool POOL1 100.0.0.1 100.0.0.2 netmask 255.255.255.0
ip nat inside source list 1 pool POOL1
```

Sekarang kita dapat menambahkan konfigurasi PAT tanpa langsung menghapus konfigurasi sebelumnya:


```
R1(config)#ip nat inside source list 1 int g0/0 overload

R1(config)#do sh run | include nat

 ip nat outside
 ip nat inside
ip nat pool POOL1 100.0.0.1 100.0.0.2 netmask 255.255.255.0
ip nat inside source list 1 pool POOL1
ip nat inside source list 1 interface GigabitEthernet0/0 overload

R1(config)#
```

Perhatikan bagian:

ip nat inside source list 1 interface GigabitEthernet0/0 overload

Artinya:

Traffic dari source yang diizinkan oleh ACL 1 akan diterjemahkan menggunakan IP interface G0/0, dan overload memungkinkan banyak perangkat menggunakan IP tersebut secara bersamaan dengan membedakan nomor port.

Berbeda dengan Dynamic NAT sebelumnya:

```
Dynamic NAT
192.168.0.1 → 100.0.0.1
192.168.0.2 → 100.0.0.2
```
PAT:

```
192.168.0.1 ─┐
192.168.0.2 ─┼──→ 203.0.113.1
192.168.0.3 ─┘
```
Jadi beberapa perangkat dapat berbagi satu IP publik.

Catatan: Jika tujuan kita benar-benar mengganti Dynamic NAT menjadi PAT, konfigurasi Dynamic NAT sebelumnya sebaiknya dihapus agar tidak ada dua aturan NAT yang aktif sekaligus. Pada lab ini kita menambahkan konfigurasi PAT untuk melihat cara kerjanya.

---

## 4. Uji Coba PAT

Sekarang kita coba melakukan ping dari PC1, PC2, dan PC3.

Contoh dari PC3:

```
Cisco Packet Tracer PC3 Command Line 1.0
C:\>ping google.com

Pinging 8.8.8.8 with 32 bytes of data:

Reply from 8.8.8.8: bytes=32 time=11ms TTL=126
Reply from 8.8.8.8: bytes<1ms TTL=126
Reply from 8.8.8.8: bytes<1ms TTL=126
Reply from 8.8.8.8: bytes<1ms TTL=126

Ping statistics for 8.8.8.8:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 0ms, Maximum = 11ms, Average = 2ms
```
Ping berhasil.

Selanjutnya kita periksa tabel translasi NAT:

```
R1(config)#do sh ip nat tr

Pro  Inside global     Inside local       Outside local      Outside global
icmp 203.0.113.1:33    192.168.0.1:33     8.8.8.8:33         8.8.8.8:33
icmp 203.0.113.1:34    192.168.0.1:34     8.8.8.8:34         8.8.8.8:34
icmp 203.0.113.1:35    192.168.0.1:35     8.8.8.8:35         8.8.8.8:35
icmp 203.0.113.1:36    192.168.0.1:36     8.8.8.8:36         8.8.8.8:36
icmp 203.0.113.1:5     192.168.0.2:5      8.8.8.8:5          8.8.8.8:5
icmp 203.0.113.1:6     192.168.0.2:6      8.8.8.8:6          8.8.8.8:6
icmp 203.0.113.1:7     192.168.0.2:7      8.8.8.8:7          8.8.8.8:7
icmp 203.0.113.1:8     192.168.0.2:8      8.8.8.8:8          8.8.8.8:8
udp 203.0.113.1:1027   192.168.0.1:1027   8.8.8.8:53         8.8.8.8:53
```
Dari hasil tersebut terlihat bahwa beberapa perangkat internal menggunakan IP yang sama, yaitu 203.0.113.1.

Contohnya:

```
192.168.0.1 → 203.0.113.1:33
192.168.0.2 → 203.0.113.1:5
```
Nomor setelah tanda : digunakan untuk membedakan koneksi.

Jadi meskipun beberapa perangkat menggunakan satu IP publik yang sama, router tetap dapat mengetahui koneksi tersebut milik perangkat mana.

### Dynamic NAT vs PAT

| Fitur                | Dynamic NAT                                 | PAT                                   |
| -------------------- | ------------------------------------------- | ------------------------------------- |
| Pemetaan             | Private → Public                            | Banyak Private → satu Public          |
| Jumlah IP publik     | Membutuhkan beberapa IP publik              | Bisa menggunakan satu IP publik       |
| Identifikasi koneksi | Berdasarkan IP                              | Berdasarkan IP + port                 |
| Contoh               | `192.168.0.1 → 100.0.0.1`                   | `192.168.0.1 → 203.0.113.1:33`        |
| Efisiensi IP publik  | Lebih rendah                                | Sangat tinggi                         |
| Penggunaan umum      | Situasi yang membutuhkan beberapa IP publik | Akses Internet untuk banyak perangkat |

Kesimpulan

Dynamic NAT menggunakan NAT Pool untuk memberikan IP publik secara otomatis kepada perangkat internal.

Pada percobaan:

```
192.168.0.1 → 100.0.0.1
192.168.0.2 → 100.0.0.2
```

Sedangkan PAT menggunakan satu IP publik dan membedakan koneksi menggunakan nomor port:



```
192.168.0.1 ─┐
192.168.0.2 ─┼──→ 203.0.113.1
192.168.0.3 ─┘
```


Inilah alasan PAT sangat umum digunakan pada jaringan rumah, kantor, dan jaringan yang memiliki banyak perangkat tetapi hanya memiliki sedikit alamat IP publik.



