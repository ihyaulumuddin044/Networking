# SSH (Secure Shell)

**SSH (Secure Shell)** adalah protokol jaringan yang digunakan untuk membuat sambungan yang aman dan terenkripsi antara dua komputer atau perangkat. SSH banyak digunakan untuk mengontrol perangkat atau server dari jarak jauh dan menjadi alternatif yang lebih aman dibandingkan **Telnet**, karena komunikasi SSH dienkripsi.

## Fungsi Utama SSH

* **Akses Jarak Jauh:** Masuk ke komputer atau perangkat lain melalui CLI.
* **Kirim File:** Memindahkan data secara aman menggunakan SCP atau SFTP.
* **Mengatur Perangkat Jaringan:** Mengelola router, switch, server, dan perangkat jaringan lainnya secara remote.

## Cara Kerja Singkat

* Secara default menggunakan **TCP port 22**.
* Mengenkripsi komunikasi sehingga data yang dikirimkan tidak mudah dibaca oleh pihak lain.
* Menggunakan autentikasi seperti **password** atau **SSH key** untuk memastikan identitas pengguna.

---

# Studi Kasus

<img width="790" height="368" alt="Screenshot 2026-08-20 143911" src="https://github.com/user-attachments/assets/2ff6a3b3-59d0-4df6-a3d4-bd97f1e6eeb5" />

*(Gambar Topologi)*

Kasus kali ini adalah kita menambahkan switch baru, yaitu **SW2**, ke dalam jaringan. SW2 belum dikonfigurasi sehingga kita akan melakukan konfigurasi dari awal.

Tujuan akhirnya adalah membuat **SW2 dapat dikonfigurasi secara remote menggunakan SSH**, sehingga kita tidak perlu lagi menggunakan kabel console setiap kali ingin mengakses switch.

---

# 1. Menyambungkan Laptop dengan SW2

## a. Menggunakan Kabel Console

Karena SW2 belum dikonfigurasi, langkah pertama adalah menghubungkan laptop secara langsung menggunakan kabel console.

Sambungkan:

```text
Laptop → RS-232 → Console SW2
```


<img width="167" height="218" alt="leptop to switch" src="https://github.com/user-attachments/assets/e528e803-ddf4-4d48-b585-62a88be4df60" />

*(Gambar Laptop ke Switch)*

Setelah terhubung, buka aplikasi **Terminal** pada laptop untuk mulai melakukan konfigurasi SW2.

---

## b. Konfigurasi SW2

Parameter yang akan digunakan:

| Parameter       | Nilai             |
| --------------- | ----------------- |
| Hostname        | `SW2`             |
| Enable Secret   | `ciscossh`        |
| Username        | `sshlab`          |
| Password User   | `ciscossh`        |
| VLAN Management | VLAN 10           |
| IP SVI VLAN 10  | `192.168.2.10/24` |
| Default Gateway | `192.168.2.1`     |

Konfigurasikan SW2:

```text
Switch(config)#hostname SW2
SW2(config)#enable secret ciscossh
SW2(config)#username sshlab secret ciscossh
SW2(config)#interface vlan 10
SW2(config-if)#
%LINK-5-CHANGED: Interface Vlan10, changed state to up

SW2(config-if)#ip address 192.168.2.10 255.255.255.0
SW2(config-if)#no shutdown
SW2(config-if)#exit
SW2(config)#ip default-gateway 192.168.2.1
```

### Penjelasan

| Command                                                  | Penjelasan                                                                          | Fungsi dalam SSH/Management                                                                                                 |
| -------------------------------------------------------- | ----------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| `hostname SW2`                                           | Mengubah nama perangkat menjadi `SW2`.                                              | Menjadi identitas switch. Nama ini juga digunakan sebagai bagian dari identitas perangkat ketika membuat RSA key untuk SSH. |
| `enable secret ciscossh`                                 | Membuat password untuk masuk ke **privileged EXEC mode** (`SW2#`).                  | Mengamankan akses ke mode privileged. **Bukan password SSH.**                                                               |
| `username sshlab secret ciscossh`                        | Membuat user lokal bernama `sshlab` dengan password `ciscossh`.                     | User ini nantinya digunakan untuk login SSH ketika VTY menggunakan `login local`.                                           |
| `interface vlan 10`                                      | Masuk ke interface virtual VLAN 10 atau **SVI (Switched Virtual Interface)**.       | SVI digunakan untuk memberikan IP management kepada switch.                                                                 |
| `%LINK-5-CHANGED: Interface Vlan10, changed state to up` | Cisco memberi tahu bahwa interface VLAN 10 mengalami perubahan status menjadi `up`. | Menandakan SVI VLAN 10 aktif secara logis.                                                                                  |
| `ip address 192.168.2.10 255.255.255.0`                  | Memberikan IP `192.168.2.10/24` kepada SVI VLAN 10.                                 | IP ini nantinya digunakan untuk mengakses SW2 secara remote, misalnya melalui SSH.                                          |
| `no shutdown`                                            | Mengaktifkan interface VLAN 10 secara administratif.                                | Memastikan SVI tidak berada dalam kondisi `administratively down`.                                                          |
| `exit`                                                   | Keluar dari konfigurasi interface VLAN 10.                                          | Kembali ke global configuration mode.                                                                                       |
| `ip default-gateway 192.168.2.1`                         | Menentukan default gateway untuk switch.                                            | Memungkinkan switch mengirim traffic management menuju jaringan di luar subnet `192.168.2.0/24`.                            |

> **Catatan:** Pada switch Layer 2, `ip default-gateway` digunakan untuk menentukan gateway bagi traffic management seperti SSH. Ini berbeda dengan konfigurasi `ip route` yang biasanya digunakan pada perangkat Layer 3.

---

# 2. Konfigurasi Line Console Security pada SW2

**Line console** merupakan konfigurasi yang digunakan untuk mengatur akses langsung ke router atau switch melalui kabel console.

Kita akan menggunakan local user database yang sebelumnya telah dibuat.

Konfigurasikan:

```text
SW2(config)#line console 0
SW2(config-line)#login local
SW2(config-line)#exec-timeout 5 0
```

### Penjelasan

| Command        | Fungsi                                                                                                       | Contoh             |
| -------------- | ------------------------------------------------------------------------------------------------------------ | ------------------ |
| `login local`  | Meminta username dan password dari **local user database** yang telah dibuat menggunakan command `username`. | `login local`      |
| `exec-timeout` | Menentukan berapa lama sesi boleh idle atau tidak ada aktivitas sebelum sesi otomatis diputus.               | `exec-timeout 5 0` |

Dengan konfigurasi tersebut, ketika kita keluar dari sesi console kemudian melakukan login kembali, kita harus memasukkan **username dan password** yang telah dibuat sebelumnya.

Setelah berhasil masuk, kita masih perlu memasukkan `enable secret` untuk masuk ke **Privileged EXEC Mode**.

<img width="706" height="585" alt="sec line console" src="https://github.com/user-attachments/assets/aa8a3fd6-e527-4d3f-acd4-e1e45db97cfe" />

*(Gambar Security Line Console)*

Masukkan username dan password yang telah dikonfigurasi sebelumnya.

---

# 3. Mengonfigurasi SW2 agar Dapat Diakses secara Remote Menggunakan SSH

Sekarang kita akan mengonfigurasi SSH pada SW2.

Parameter yang digunakan:

| Parameter      | Nilai        |
| -------------- | ------------ |
| Domain Name    | `sshlab.com` |
| RSA Key Size   | `2048 bit`   |
| Authentication | Local User   |
| Timeout        | 5 menit      |
| Protocol       | SSH only     |
| Limit Access   | PC0 only     |

Konfigurasi:

```text
SW2(config)#ip domain name sshlab.com
SW2(config)#crypto key generate rsa
The name for the keys will be: SW2.sshlab.com
Choose the size of the key modulus in the range of 360 to 4096 for your
  General Purpose Keys. Choosing a key modulus greater than 512 may take
  a few minutes.

How many bits in the modulus [512]: 2048
% Generating 2048 bit RSA keys, keys will be non-exportable...[OK]

*Mar 1 1:27:4.668: %SSH-5-ENABLED: SSH 1.99 has been enabled

SW2(config)#access-list 1 permit host 192.168.1.10
SW2(config)#line vty 0 15
SW2(config-line)#login local
SW2(config-line)#exec-timeout 5 0
SW2(config-line)#transport input ssh
SW2(config-line)#access-class 1 in
```

### Penjelasan

| Requirement                    | Konfigurasi                        | Status | Penjelasan                                                                                                  |
| ------------------------------ | ---------------------------------- | :----: | ----------------------------------------------------------------------------------------------------------- |
| **Domain name: `sshlab.com`**  | `ip domain name sshlab.com`        |    ✅   | Menentukan domain name perangkat. Digunakan sebagai bagian dari identitas perangkat ketika membuat RSA key. |
| **RSA key size: 2048**         | `crypto key generate rsa` → `2048` |    ✅   | Membuat RSA key berukuran 2048-bit yang digunakan dalam proses kriptografi SSH.                             |
| **Authentication: local user** | `login local`                      |    ✅   | SSH menggunakan username/password yang dibuat melalui `username ... secret ...`.                            |
| **Timeout: 5 menit**           | `exec-timeout 5 0`                 |    ✅   | Sesi yang idle selama 5 menit akan otomatis diputus.                                                        |
| **Protocol: SSH only**         | `transport input ssh`              |    ✅   | VTY hanya menerima koneksi SSH. Telnet tidak diizinkan.                                                     |
| **Limit access: PC0 only**     | `access-class 1 in` + ACL          |    ✅   | Hanya source IP `192.168.1.10` yang diizinkan mengakses VTY.                                                |

### Memahami ACL pada VTY

Kita membuat Standard ACL:

```text
SW2(config)#access-list 1 permit host 192.168.1.10
```

Artinya, hanya perangkat dengan source IP:

```text
192.168.1.10
```

yang diizinkan mengakses VTY.

Kemudian ACL tersebut diterapkan pada VTY:

```text
SW2(config-line)#access-class 1 in
```

`in` berarti ACL memeriksa **traffic yang masuk menuju VTY**.

Dengan demikian, meskipun perangkat lain mengetahui alamat IP management SW2, hanya PC0 dengan IP `192.168.1.10` yang dapat melakukan koneksi SSH.

> **Catatan:** Standard ACL memiliki implicit `deny any` di akhir ACL. Karena ACL nomor 1 hanya memiliki `permit host 192.168.1.10`, source IP lainnya secara otomatis ditolak.

---

# 4. Uji Coba Konfigurasi

Sekarang kita akan menguji konfigurasi dengan melakukan **ping dan SSH** dari R2 menuju SW2.

## Pengujian dari R2

Lakukan ping:

```text
R2#ping 192.168.2.10

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.2.10, timeout is 2 seconds:
.....
Success rate is 0 percent (0/5)
```

Ping gagal.

Setelah melakukan troubleshooting, ditemukan bahwa kita belum mengonfigurasi satu pun port pada SW2 sebagai **access port untuk VLAN 10**.

Padahal SVI VLAN 10 membutuhkan VLAN 10 yang benar-benar aktif pada switch. Karena belum ada port yang menjadi anggota VLAN 10 dan aktif, SVI belum dapat berfungsi sebagaimana mestinya.

Kita akan menggunakan kabel dari R2 menuju:

```text
R2 → GigabitEthernet0/1 SW2
```

Sebelum memperbaikinya, kita dapat melihat kondisi VLAN menggunakan:

```text
SW2#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/1, Fa0/2, Fa0/3, Fa0/4
                                                Fa0/5, Fa0/6, Fa0/7, Fa0/8
                                                Fa0/9, Fa0/10, Fa0/11, Fa0/12
                                                Fa0/13, Fa0/14, Fa0/15, Fa0/16
                                                Fa0/17, Fa0/18, Fa0/19, Fa0/20
                                                Fa0/21, Fa0/22, Fa0/23, Fa0/24
                                                Gig0/1, Gig0/2
10   VLAN0010                         active
1002 fddi-default                     active
1003 token-ring-default               active
1004 fddinet-default                  active
1005 trnet-default                     active
```

Terlihat bahwa **VLAN 10 belum memiliki port anggota**.

Mari kita perbaiki dengan memasukkan GigabitEthernet0/1 ke VLAN 10 sebagai access port:

```text
SW2(config)#interface g0/1
SW2(config-if)#switchport mode access
SW2(config-if)#switchport access vlan 10
%LINEPROTO-5-UPDOWN: Line protocol on Interface Vlan10, changed state to up

SW2(config-if)#no shutdown
```

### Penjelasan

| Command / Output                                             | Penjelasan                                                                    | Efek                                                                         |
| ------------------------------------------------------------ | ----------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| `interface g0/1`                                             | Masuk ke interface fisik `GigabitEthernet0/1`.                                | Semua konfigurasi berikutnya diterapkan pada Gi0/1.                          |
| `switchport mode access`                                     | Mengubah Gi0/1 menjadi **access port**.                                       | Port digunakan untuk satu VLAN, bukan membawa banyak VLAN seperti trunk.     |
| `switchport access vlan 10`                                  | Memasukkan Gi0/1 ke **VLAN 10**.                                              | Traffic yang masuk melalui Gi0/1 menjadi bagian dari VLAN 10.                |
| `%LINEPROTO-5-UPDOWN: Interface Vlan10, changed state to up` | Cisco memberi tahu bahwa Line Protocol pada SVI VLAN 10 berubah menjadi `up`. | SVI VLAN 10 sekarang dapat beroperasi karena VLAN 10 memiliki port aktif. 🎯 |
| `no shutdown`                                                | Mengaktifkan interface Gi0/1 secara administratif.                            | Port tidak berada dalam kondisi `administratively down`.                     |

> **Catatan:** SVI VLAN 10 dapat menjadi `up/up` ketika VLAN tersebut aktif dan memiliki setidaknya satu port yang aktif secara fisik dan berada pada VLAN tersebut. Inilah alasan penambahan Gi0/1 ke VLAN 10 menyelesaikan masalah konektivitas management pada lab ini.

---

## Pengujian Kembali dari R2

Sekarang kita kembali ke R2 dan melakukan ping:

```text
R2#ping 192.168.2.10

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.2.10, timeout is 2 seconds:
..!!!
Success rate is 60 percent (3/5), round-trip min/avg/max = 0/0/0 ms
```

Ping sudah berhasil.

Selanjutnya kita mencoba melakukan SSH:

```text
R2#ssh -l sshlab 192.168.2.10

% Connection refused by remote host
```

SSH dari R2 ditolak.

Hal ini justru menunjukkan bahwa **ACL pada VTY bekerja sesuai dengan konfigurasi**.

R2 bukan merupakan source IP yang diizinkan oleh ACL:

```text
access-list 1 permit host 192.168.1.10
```

Sehingga koneksi SSH dari R2 ditolak.

---

# Pengujian dari PC0

Sekarang kita mencoba melakukan koneksi dari **PC0**, yaitu perangkat yang memiliki alamat IP yang telah diizinkan oleh ACL:

```text
192.168.1.10
```

<img width="667" height="559" alt="Screenshot 2026-08-20 143622" src="https://github.com/user-attachments/assets/7caf444f-383c-49ea-b940-358212e27f4c" />

*(Gambar Ping dan SSH dari PC0)*

PC0 berhasil melakukan koneksi ke SW2 menggunakan SSH.

Dengan demikian, konfigurasi SSH dan pembatasan akses menggunakan Standard ACL telah berhasil diterapkan.

PC0:

```text
192.168.1.10
        │
        │ SSH
        ▼
      SW2
192.168.2.10
```

Sedangkan perangkat lain yang tidak termasuk dalam ACL akan ditolak ketika mencoba mengakses VTY melalui SSH.

---

# Kesimpulan

Pada praktik ini kita telah mempelajari cara mengonfigurasi **SSH pada switch Cisco** agar perangkat dapat dikelola secara remote tanpa menggunakan kabel console.

Beberapa konfigurasi utama yang telah dilakukan:

1. Membuat hostname dan `enable secret`.
2. Membuat local user untuk autentikasi SSH.
3. Membuat **SVI VLAN 10** sebagai IP management switch.
4. Mengonfigurasi default gateway.
5. Mengamankan akses console menggunakan `login local`.
6. Membuat domain name dan **RSA key 2048-bit**.
7. Mengonfigurasi VTY agar hanya menerima **SSH**.
8. Menggunakan `login local` untuk autentikasi user.
9. Mengatur `exec-timeout` selama 5 menit.
10. Menggunakan Standard ACL untuk membatasi akses SSH hanya dari PC0.
11. Melakukan troubleshooting ketika SVI VLAN 10 belum aktif.
12. Menguji koneksi menggunakan ping dan SSH.

Dengan konfigurasi tersebut, SW2 sekarang dapat dikelola secara remote menggunakan **SSH**, sementara akses SSH dibatasi hanya untuk perangkat yang memiliki source IP yang diizinkan oleh ACL.
