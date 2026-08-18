# Syslog

**Syslog (System Logging)** adalah mekanisme standar untuk menghasilkan, menyimpan, dan mengirimkan pesan log dari berbagai perangkat, seperti:

* Router
* Switch
* Firewall
* Server
* Access Point
* Perangkat jaringan lainnya

Contohnya ketika interface router mati:

```text
%LINK-3-UPDOWN: Interface GigabitEthernet0/0, changed state to down
```

Router memberikan informasi:

> "Interface GigabitEthernet0/0 sekarang DOWN."

Jadi, Syslog membantu administrator mengetahui apa yang terjadi pada perangkat, bahkan ketika administrator tidak sedang melihat atau mengakses perangkat tersebut.

---

## Severity Level pada Syslog

Syslog memiliki **8 severity level**, mulai dari level **0 sampai 7**.

| Level | Severity      | Gampangnya                      |
| ----: | ------------- | ------------------------------- |
| **0** | Emergency     | ☠️ Sistem tidak dapat digunakan |
| **1** | Alert         | 🚨 Harus segera ditangani       |
| **2** | Critical      | 🔥 Kondisi kritis               |
| **3** | Error         | ❌ Terjadi error                 |
| **4** | Warning       | ⚠️ Peringatan                   |
| **5** | Notice        | 📢 Kondisi penting              |
| **6** | Informational | ℹ️ Informasi                    |
| **7** | Debug         | 🔍 Informasi debugging          |

**Semakin kecil angka severity, semakin serius kondisi yang dilaporkan.**

---

## Cara Membaca Syslog Cisco

Ini merupakan bagian yang sangat penting untuk dipahami.

Misalnya terdapat pesan:

```text
%LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to down
```

Format sederhananya:

```text
%FACILITY-SEVERITY-MNEMONIC: MESSAGE
```

Kita bedah satu per satu:

```text
%LINK-3-UPDOWN
  │    │   │
  │    │   └── Mnemonic
  │    └────── Severity
  └─────────── Facility
```

### 1. Facility

```text
LINK
```

Menunjukkan **facility**, yaitu bagian atau subsistem perangkat yang menghasilkan pesan tersebut.

### 2. Severity

```text
3
```

Menunjukkan tingkat keparahan pesan.

Dalam contoh ini:

```text
3 = Error
```

### 3. Mnemonic

```text
UPDOWN
```

Mnemonic adalah kode singkat yang menggambarkan jenis kejadian yang terjadi.

### 4. Message

```text
Interface GigabitEthernet0/1, changed state to down
```

Bagian ini menjelaskan kejadian sebenarnya secara lebih detail.

Jadi, pesan tersebut dapat dibaca sebagai:

> **Facility LINK mengalami kejadian dengan severity level 3 (Error), yaitu interface GigabitEthernet0/1 berubah menjadi down.**

---

# Studi Kasus

<img width="579" height="443" alt="Screenshot 2026-08-18 185736" src="https://github.com/user-attachments/assets/de7cae9d-67f8-454e-b2eb-b84e69db7adb" />

## 1. Melihat Syslog melalui Console

Pertama, kita akan melakukan koneksi ke router menggunakan **PC1 melalui console**.

Sambungkan:

```text
PC → RS-232 → Router → Console
```

Kemudian buka aplikasi **Terminal** pada PC.

<img width="838" height="584" alt="console ke R1" src="https://github.com/user-attachments/assets/55b492fa-bcce-4cc1-97e2-331af2482060" />
*(Gambar koneksi Console ke R1)*

Pada kasus ini, setelah kita memberikan alamat IP dan mengaktifkan interface, muncul pesan Syslog seperti berikut:

```text
Router(config-if)#
%LINK-5-CHANGED: Interface GigabitEthernet0/1, changed state to up

%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to up
```

Pesan pertama:

```text
%LINK-5-CHANGED: Interface GigabitEthernet0/1, changed state to up
```

menjelaskan bahwa interface **GigabitEthernet0/1** mengalami perubahan status menjadi **up**.

Severity yang digunakan adalah:

```text
5 = Notice
```

Sedangkan `CHANGED` merupakan mnemonic yang menunjukkan bahwa terjadi perubahan status.

Pesan kedua:

```text
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to up
```

menunjukkan bahwa **Line Protocol** pada interface GigabitEthernet0/1 juga telah berubah menjadi **up**.

Secara sederhana, setelah kabel terhubung dan konfigurasi interface benar, perangkat pada kedua ujung berhasil membangun koneksi dan jalur komunikasi siap digunakan untuk lalu lintas jaringan.

---

## Memberikan Timestamp pada Log

Pada contoh sebelumnya, tidak terdapat informasi waktu yang jelas mengenai kapan suatu perubahan atau masalah terjadi.

Hal ini dapat menyulitkan administrator ketika melakukan **troubleshooting** dan **monitoring**, terutama ketika harus mengetahui urutan kejadian.

Oleh karena itu, kita dapat menambahkan timestamp pada pesan Syslog dengan perintah:

```text
Router(config)#service timestamps log datetime
% Incomplete command.
Router(config)#service timestamps log datetime msec
```

Pada percobaan pertama kita mendapatkan pesan:

```text
% Incomplete command.
```

Hal ini terjadi karena pada **Cisco Packet Tracer**, beberapa command memiliki implementasi yang berbeda dibandingkan perangkat Cisco asli. Pada Packet Tracer, kita perlu melengkapi command dengan opsi `msec`.

Setelah konfigurasi selesai, ketika kita keluar dari mode konfigurasi atau menggunakan `Ctrl+C`, akan muncul pesan seperti:

```text
*Mar 01, 00:45:58.4545: %SYS-5-CONFIG_I: Configured from console by console
```

Sekarang pesan Syslog sudah memiliki **timestamp** sehingga kita dapat mengetahui kapan pesan tersebut terjadi.

> **Catatan**
>
> Waktu yang ditampilkan pada simulasi belum tentu akurat karena perangkat belum dikonfigurasi menggunakan waktu yang benar. Karena pembahasan ini berfokus pada pengenalan Syslog, konfigurasi waktu dapat dibahas pada materi berikutnya.

---

# 2. Melakukan Telnet ke Router

Selanjutnya kita akan mencoba mengakses router secara remote menggunakan **Telnet**.

Pada **Command Prompt PC2**, jalankan:

```text
C:\>telnet 192.168.1.1
Trying 192.168.1.1 ...Open

[Connection to 192.168.1.1 closed by foreign host]
```

Koneksi Telnet berhasil mencapai router, tetapi router langsung memutus koneksi.

Hal ini terjadi karena **VTY (Virtual Terminal Lines)** belum dikonfigurasi dengan benar.

VTY merupakan jalur virtual yang digunakan perangkat Cisco untuk menyediakan akses remote seperti **Telnet** dan **SSH**.

Untuk mengonfigurasi VTY, gunakan:

```text
Router(config)#line vty 0 4
Router(config-line)#password cisco
Router(config-line)#login
Router(config-line)#transport input telnet
```

| Command                  | Bagian            | Fungsi                                                                                                                                                     |
| ------------------------ | ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `line vty 0 4`           | `line vty`        | Masuk ke konfigurasi **Virtual Terminal Line (VTY)**, yaitu jalur virtual untuk akses remote seperti Telnet/SSH.                                           |
| `0 4`                    | Nomor line        | Memilih VTY line 0 sampai 4. Artinya terdapat 5 line virtual yang menggunakan konfigurasi tersebut.                                                        |
| `password cisco`         | `password`        | Menentukan password yang digunakan untuk autentikasi pada VTY.                                                                                             |
| `cisco`                  | Password          | Password VTY yang ditentukan. Dalam lab dapat menggunakan `cisco`, tetapi pada jaringan nyata sebaiknya menggunakan mekanisme autentikasi yang lebih aman. |
| `login`                  | `login`           | Memerintahkan router untuk meminta password ketika seseorang mencoba masuk melalui VTY.                                                                    |
| `transport input telnet` | `transport input` | Menentukan protokol yang diizinkan masuk melalui VTY.                                                                                                      |
| `telnet`                 | Protokol          | Hanya Telnet yang diizinkan pada VTY tersebut.                                                                                                             |

Setelah konfigurasi selesai, PC2 sudah dapat melakukan koneksi remote ke router menggunakan Telnet.

<img width="244" height="142" alt="PC2 telnet ke r1" src="https://github.com/user-attachments/assets/aae59d78-dfbd-40b4-b316-1d56abb88e78" />

*(Gambar PC2 Telnet ke Router)*

---

## Menguji Syslog melalui Telnet

Sekarang kita akan mencoba melihat informasi interface dan melakukan konfigurasi router melalui PC2 menggunakan Telnet.

Jalankan:

```text
Router#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     unassigned      YES unset  administratively down down
GigabitEthernet0/1     192.168.1.1     YES manual up                    up
GigabitEthernet0/2     unassigned      YES unset  administratively down down
Vlan1                  unassigned      YES unset  administratively down down

Router#configure terminal
Enter configuration commands, one per line.  End with CNTL/Z.
Router(config)#interface g0/0
Router(config-if)#no shutdown

Router(config-if)#
```

Meskipun R1 sudah menjalankan `no shutdown`, kita mungkin tidak melihat pesan Syslog pada sesi Telnet.

Kenapa?

Karena **Telnet bukan Syslog Server**.

Telnet hanya menyediakan terminal remote untuk mengakses CLI router. Agar pesan Syslog dapat ditampilkan pada sesi terminal tersebut, kita harus mengaktifkan monitoring terminal.

Gunakan:

```text
Router(config-if)#do terminal monitor
```

Sekarang coba matikan kembali interface GigabitEthernet0/0:

```text
Router(config-if)#interface g0/0
Router(config-if)#shutdown

Router(config-if)#
*Mar 01, 01:20:11.2020: %LINK-5-CHANGED: Interface GigabitEthernet0/0, changed state to administratively down
```

Pesan Syslog sekarang muncul pada sesi Telnet.

> **Catatan**
>
> `terminal monitor` berlaku pada **sesi terminal yang sedang aktif**. Jika kita keluar dari sesi Telnet kemudian masuk kembali, kita perlu mengaktifkan `terminal monitor` lagi pada sesi yang baru.

---

# 3. Mengaktifkan Logging ke Buffer

Secara default, buffer logging pada perangkat belum dikonfigurasi untuk menyimpan pesan log dalam ukuran tertentu.

**Buffer** adalah area penyimpanan sementara yang digunakan perangkat untuk menampung data sebelum data tersebut diproses, dikirim, atau ditampilkan.

Jika kita ingin router menyimpan pesan Syslog ke dalam buffer, kita dapat mengaktifkannya dengan:

```text
Router(config)#logging buffered 8192
Router(config)#
*Mar 01, 01:26:14.2626: SYS-5-LOG_CONFIG_CHANGE: Buffer logging: level debugging, xml disabled, filtering disabled, size (8192)
```

Angka:

```text
8192
```

menunjukkan ukuran buffer yang digunakan untuk menyimpan pesan log.

Dengan konfigurasi tersebut, router dapat menyimpan pesan Syslog secara lokal di dalam buffer sehingga administrator dapat melihat kembali log yang masih tersimpan.

Untuk melihat isi buffer, kita dapat menggunakan:

```text
Router#show logging
```

Command tersebut akan menampilkan informasi logging, termasuk pesan-pesan Syslog yang tersimpan pada buffer.

> **Catatan**
>
> Buffer bersifat **sementara**. Jika perangkat mengalami reboot, isi buffer dapat hilang. Karena itu, pada lingkungan jaringan nyata biasanya log juga dikirimkan ke Syslog Server agar penyimpanannya lebih persisten.

---

# 4. Mengirim Logging ke Syslog Server

Karena buffer memiliki kapasitas terbatas dan bersifat sementara, kita membutuhkan tempat penyimpanan log yang lebih persisten.

Salah satu solusinya adalah mengirimkan pesan Syslog dari router ke **Syslog Server**.

Ada dua command penting yang perlu dipahami:

```text
logging host
```

dan

```text
logging trap
```

Jika:

```text
logging host
```

menentukan **ke mana** log dikirim,

maka:

```text
logging trap
```

menentukan **severity level berapa** yang akan dikirim.

Konfigurasikan Syslog Server dengan:

```text
Router(config)#logging host 192.168.1.100
Router(config)#logging trap ?
  debugging  Debugging messages                (severity=7)

Router(config)#logging trap debugging
```

Pada konfigurasi tersebut:

```text
192.168.1.100
```

merupakan alamat IP Syslog Server.

Sedangkan:

```text
logging trap debugging
```

menentukan level logging yang dikirim ke Syslog Server.

Setelah konfigurasi selesai, keluar dari mode konfigurasi dan lakukan beberapa aktivitas pada router.

Kemudian buka **Server → Syslog** pada Cisco Packet Tracer untuk melihat pesan log yang diterima.

<img width="829" height="559" alt="hasil syslog" src="https://github.com/user-attachments/assets/f298d34f-da4c-4336-a6b8-f0e8a164d164" />

*(Gambar hasil Syslog Server)*

---

## Severity pada `logging trap`

Pada perangkat Cisco asli, kita dapat menentukan severity level tertentu menggunakan angka.

Contohnya:

```text
R1(config)#logging trap 4
```

Artinya:

> Kirim pesan Syslog dengan severity level **0 sampai 4** ke Syslog Server yang telah dikonfigurasi.

Karena severity level semakin kecil berarti semakin serius, maka konfigurasi level `4` akan mengirim:

```text
0 Emergency
1 Alert
2 Critical
3 Error
4 Warning
```

Sedangkan pesan dengan severity level 5, 6, dan 7 tidak dikirim berdasarkan konfigurasi tersebut.

> **Catatan Cisco Packet Tracer**
>
> Dalam simulasi Cisco Packet Tracer yang digunakan pada lab ini, opsi `logging trap` yang tersedia terbatas pada level:
>
> ```text
> debugging
> ```
>
> atau severity level **7**.
>
> Hal ini merupakan keterbatasan simulasi Packet Tracer dan tidak sepenuhnya mencerminkan kemampuan perangkat Cisco asli.

---

# Kesimpulan

Syslog merupakan salah satu mekanisme penting dalam **monitoring dan troubleshooting jaringan**. Dengan Syslog, administrator dapat mengetahui berbagai kejadian yang terjadi pada router, switch, firewall, server, dan perangkat jaringan lainnya.

Pada praktik ini kita telah mempelajari beberapa cara kerja logging pada Cisco:

1. Melihat pesan Syslog langsung melalui **Console**.
2. Menambahkan **timestamp** pada pesan log.
3. Mengakses router menggunakan **Telnet**.
4. Menampilkan Syslog pada sesi remote menggunakan `terminal monitor`.
5. Menyimpan log sementara menggunakan **logging buffer**.
6. Mengirim log ke **Syslog Server** menggunakan `logging host`.
7. Mengatur severity log yang dikirim menggunakan `logging trap`.

Dengan memahami Syslog, kita tidak hanya dapat melihat bahwa "sesuatu terjadi" pada perangkat, tetapi juga dapat mengetahui **kapan kejadian tersebut terjadi, bagian perangkat mana yang terpengaruh, seberapa serius masalahnya, dan apa pesan yang menjelaskan kejadian tersebut**.
