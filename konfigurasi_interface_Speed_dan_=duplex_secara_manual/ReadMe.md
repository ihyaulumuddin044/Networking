# ⚙️ Konfigurasi Interface, Speed, dan Duplex Secara Manual

Dokumentasi ini membahas konfigurasi dasar pada perangkat Cisco yang meliputi:

- Pemberian hostname dan password
- Konfigurasi IP address pada router
- Pengaturan speed dan duplex secara manual
- Pemberian deskripsi interface
- Konfigurasi IP pada host (PC)
- Penyimpanan konfigurasi

Praktik ini dapat dilakukan menggunakan **Cisco Packet Tracer**, **GNS3**, atau perangkat Cisco asli.

---

## 1️⃣ Memberikan Hostname dan Password

### 🔐 Password EXEC Mode


<img width="446" height="66" alt="secret cisco" src="https://github.com/user-attachments/assets/d2d6492c-0c99-45ba-99fd-bda31a04a6d0" />

Pada kode di atas bisa dilihat bagaimana cara melakukan pemberian sandi pada EXEC mode.  
Anda bisa menggunakan:

- `enable password`
- `enable secret`

Lebih disarankan menggunakan **`enable secret`** karena tersimpan dalam bentuk terenkripsi.

---

### 🏷 Memberikan Hostname

<img width="446" height="66" alt="memberikan hostname" src="https://github.com/user-attachments/assets/3a6bf058-0e79-4121-8a42-82abb370012d" />

Kode di atas berfungsi untuk memberikan nama pada perangkat agar mudah dalam melakukan troubleshooting atau pemeliharaan.

Contoh:

hostname R1 atau SW1, SW2 dan lainnya


---

## 2️⃣ Konfigurasi IP Address pada Router

<img width="447" height="77" alt="memeberikan alam ip dan subnet" src="https://github.com/user-attachments/assets/f9e77659-7bbc-4fc6-bfec-44e484dcdf20" />


Terlihat pada gambar bahwa digunakan interface **GigabitEthernet** yang mendukung kecepatan tinggi.

Konfigurasi ini nantinya akan dijadikan sebagai **default gateway** untuk host.

⚠️ Jangan lupa mengetik:

no shutdown

setelah melakukan konfigurasi alamat IP agar interface aktif.

---

## 3️⃣ Konfigurasi Speed dan Duplex Secara Manual

<img width="447" height="82" alt="memebrikan speed dan duplex secara manual" src="https://github.com/user-attachments/assets/d6121198-f43f-4a6c-a1bc-d345f86b06d9" />


Perintah tersebut digunakan untuk mengatur:

- Speed (kecepatan)
- Duplex (half / full)

secara manual pada interface yang terhubung satu sama lain.

---

### 🔍 Verifikasi Status Interface

<img width="542" height="134" alt="status ip di interfaces" src="https://github.com/user-attachments/assets/878a6797-96c7-4034-bc62-5bdac3d50407" />


Gunakan:

`sh ip int brief`
Jika berada di EXEC mode.
`do sh ip int brief`
Jika masih di global configuration.


Pastikan:

- IP Address sudah muncul
- Status **up**
- Protocol **up**

---

### ❗ Jika Protocol Masih Down

<img width="488" height="108" alt="jika protokol down" src="https://github.com/user-attachments/assets/d15523ca-1afa-4b02-bb30-98e1c0bdba18" />


Jika protocol masih `down`, masuk ke switch melalui CLI lalu gunakan perintah konfigurasi pada interface terkait.

---

## 4️⃣ Memberikan Deskripsi pada Setiap Interface

Deskripsi berguna untuk:

- Memudahkan troubleshooting
- Pemeliharaan
- Monitoring

---

<img width="562" height="191" alt="memberikan descripsi interfaces di router" src="https://github.com/user-attachments/assets/7d9683e7-f147-4d07-8357-273064f1ef25" />


Pada gambar di atas dilakukan pemberian keterangan pada interface.

Untuk memastikan deskripsi sudah tersimpan, gunakan:

`show running-config`


Hasilnya akan terlihat seperti berikut:

<img width="562" height="191" alt="hasil sh run di router" src="https://github.com/user-attachments/assets/d408e79b-8d3d-4b74-88f6-a84e61ac89d5" />

---

### 📟 Deskripsi pada Switch

Untuk sisi switch caranya sama.

<img width="562" height="106" alt="Screenshot from 2026-02-04 22-16-19" src="https://github.com/user-attachments/assets/57a89e5f-c85b-43f3-af27-f103bc206866" />

Lalu cek hasilnya:

<img width="564" height="115" alt="hasil sh int statsu" src="https://github.com/user-attachments/assets/affdf50b-26cf-4a0b-b75f-34e701bc7e26" />


Lakukan hal yang sama pada setiap port dan sesuaikan dengan interface yang digunakan.

---

## 5️⃣ Konfigurasi IP pada Host (PC)

Konfigurasi default gateway di PC dilakukan melalui:

Menu **Config → Settings**

Pada bagian **Gateway/DNS IPv4**, masukkan:

untuk defaul: 172.16.255.254 

IP pc 172.16.0.1 255.2550.0.0


---

## 6️⃣ Menyimpan Konfigurasi

Ingat untuk selalu menyimpan konfigurasi agar ketika device mati tidak perlu melakukan konfigurasi ulang.

Pada perangkat Cisco asli bisa menggunakan:

- `copy running-config startup-config`
- `write`

Pada simulator dapat menggunakan:

- `write`
- `wr`
- `write memory`
- `copy running-config startup-config`

---

## 📚 Catatan

Dokumen ini merupakan bagian dari dokumentasi pembelajaran jaringan komputer.

Materi akan terus dikembangkan mencakup:

- VLAN
- Routing
- NAT
- Network Security
- Troubleshooting Lanjutan

---

✍️ *Ditulis sebagai bagian dari perjalanan belajar Computer Networking.*
























