# Panduan Konfigurasi Simple Network Management Protocol (SNMP)

Dokumentasi ini berisi penjelasan konsep dasar SNMP, komponen utamanya, keterbatasan fitur simulasi pada Cisco Packet Tracer, serta praktik langsung (*hands-on*) konfigurasi *Community String* (Read-Only dan Read-Write) dan pengujian MIB Browser menggunakan perintah `Get` dan `Set`.

---

## 1. Konsep Dasar SNMP

**SNMP (Simple Network Management Protocol)** adalah protokol lapisan aplikasi (*application layer protocol*) yang digunakan untuk mengelola, memantau, dan mengatur perangkat dalam jaringan komputer seperti router, switch, dan server secara terpusat.

### 🏗️ Komponen Utama SNMP
1. **SNMP Manager:** Perangkat lunak pusat (biasanya terpasang di Network Management Station / NMS) yang bertugas meminta data dan mengontrol perangkat lain di jaringan.
2. **SNMP Agent:** Program/layanan kecil yang berjalan di dalam perangkat jaringan (router/switch) yang memantau kondisi lokal dan merespons permintaan dari SNMP Manager.
3. **MIB (Management Information Base):** Struktur basis data terorganisir yang berisi daftar variabel atau objek data dari perangkat yang diatur oleh agen.

### 🔄 Cara Kerja & Perintah Utama
* **SNMPGet:** Perintah yang dikirim dari Manager ke Agent untuk meminta/mengambil nilai data tertentu dari MIB.
* **SNMPSet:** Perintah dari Manager untuk mengubah/mengonfigurasi nilai parameter tertentu pada perangkat Agen.
* **SNMP Trap:** Pesan peringatan otomatis yang dikirim oleh Agent ke Manager jika terjadi peristiwa/masalah penting secara *unsolicited* (tanpa diminta).

> *Catatan: Untuk penjelasan arsitektur yang lebih rinci, disarankan untuk merujuk ke dokumentasi resmi RFC atau referensi tambahan.*

---

## 2. Keterbatasan SNMP pada Cisco Packet Tracer

> ⚠️ **Catatan Penting:**  
> Fitur SNMP dalam Cisco Packet Tracer memiliki keterbatasan fungsi. Sebagai contoh, saat mengecek perintah yang tersedia:
> ```routing
> R1(config)# snmp-server ?
>   community  Enable SNMP; set community string and access privs
> ```
> Opsi `community` menjadi satu-satunya perintah yang tersedia. Kita tidak dapat memilih atau menentukan alamat IP server tujuan untuk pengiriman *SNMP Trap*. Oleh karena itu, simulasi ini berfokus pada konfigurasi *Community String* dan pengujian via MIB Browser.

---

## 3. Studi Kasus & Langkah-Langkah Konfigurasi

<img width="536" height="307" alt="SNMP topologi" src="https://github.com/user-attachments/assets/62646819-01cd-4960-bc56-3dc1bfc2815a" />

*(Gambar Topologi SNMP)*

---

### Langkah 1: Konfigurasi SNMP Community pada R1

Skenario kebutuhan *community string*:
* **Read-Only (RO):** `cisco1`
* **Read-Write (RW):** `cisco2`

#### Eksekusi Perintah:
```
R1(config)# snmp-server community cisco1 ro
%SNMP-5-WARMSTART: SNMP agent on host R1 is undergoing a warm start
R1(config)# snmp-server community cisco2 rw
```
Penjelasan perintah dan sistem:

| Perintah                          | Fungsi                                                                        | Penjelasan                                                                                                                                  |
| --------------------------------- | ----------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `snmp-server community cisco1 ro` | Membuat **community string** bernama `cisco1` dengan hak **Read Only (RO)**.  | User atau software monitoring hanya bisa **membaca informasi** dari router, tidak bisa mengubah konfigurasi.                                |
| `%SNMP-5-WARMSTART...`            | Pesan sistem Cisco.                                                           | Menandakan **SNMP Agent** di router melakukan **warm start** setelah konfigurasi SNMP berubah. Ini adalah **pesan informasi**, bukan error. |
| `snmp-server community cisco2 rw` | Membuat **community string** bernama `cisco2` dengan hak **Read Write (RW)**. | User atau software monitoring dapat **membaca dan mengubah** informasi tertentu pada router melalui SNMP (jika didukung).                   |

## Langkah 2: Menggunakan Pesan SNMP **Get** melalui MIB Browser

Setelah konfigurasi SNMP pada router selesai, langkah berikutnya adalah melakukan pengujian menggunakan **MIB Browser** yang terdapat pada **PC1**. Pengujian ini bertujuan untuk memastikan bahwa PC1 dapat membaca informasi (read-only) dari router **R1** melalui protokol SNMP.

### Tujuan Pengujian

Pada pengujian ini kita akan mengambil beberapa informasi dari R1, yaitu:

- Mengetahui berapa lama router telah aktif (**System Uptime**).
- Menampilkan nama host (**Hostname**) router.
- Melihat jumlah interface yang dimiliki router beserta informasinya.

---

### Langkah-Langkah Pengujian

1. Buka aplikasi **MIB Browser** pada **PC1**.
2. Masukkan alamat IP router **R1** pada kolom **Address**.
3. Buka menu **Advanced**, kemudian isi informasi berikut:

| Parameter | Nilai |
|-----------|-------|
| Read Community | `cisco1` |
| Write Community | `cisco2` |

> **Keterangan**
>
> - **Read Community (`cisco1`)** digunakan untuk membaca informasi dari router (**Read-Only**).
> - **Write Community (`cisco2`)** digunakan apabila ingin melakukan perubahan konfigurasi melalui SNMP (**Read-Write**).

*(Gambar Konfigurasi Advanced MIB Browser)*

---

### 1. Memeriksa Uptime Router (sysUpTime)

Navigasikan ke Object Identifier (OID):

```text
sysUpTime
.1.3.6.1.2.1.1.3.0
```

Pada menu **Operation**, pilih **Get**, kemudian klik **Go**.

<img width="745" height="582" alt="Screenshot 2026-07-28 143324" src="https://github.com/user-attachments/assets/844c5de1-9bc6-4704-8ed4-2ac51c78a60a" />

*(Gambar Pengujian sysUpTime)*

#### Hasil

MIB Browser berhasil menampilkan nilai **System Uptime**, yaitu lama waktu router telah aktif sejak pertama kali dinyalakan atau sejak terakhir melakukan proses reboot.

---

### 2. Memeriksa Hostname Router (sysName)

Navigasikan ke OID berikut:

```text
sysName
.1.3.6.1.2.1.1.5.0
```

Pilih **Get**, kemudian klik **Go**.

<img width="844" height="586" alt="Screenshot 2026-07-28 144341" src="https://github.com/user-attachments/assets/9ddf15a0-f56d-405e-9990-03c18cff520d" />

*(Gambar Pengujian sysName)*

#### Hasil

MIB Browser berhasil mengambil informasi **Hostname** dari router R1 dan menampilkannya dalam bentuk string sesuai dengan nama host yang telah dikonfigurasi pada perangkat.

---

### 3. Melihat Informasi Interface Router

Untuk mengetahui jumlah interface yang dimiliki router beserta informasinya, navigasikan ke OID berikut:

```text
ifTable
.1.3.6.1.2.1.2.2
```

atau

```text
ifNumber
.1.3.6.1.2.1.2.1.0
```

Gunakan operasi **Get** untuk melihat jumlah interface, atau **Get Next**/**Walk** untuk menampilkan seluruh informasi interface yang tersedia.

#### Hasil

MIB Browser berhasil menampilkan informasi mengenai interface yang terdapat pada router R1, seperti jumlah interface, nama interface, status operasional, serta informasi lain yang tersimpan di dalam **Management Information Base (MIB)**.

> **Catatan**
>
> Operasi **Get** digunakan untuk mengambil nilai dari satu OID tertentu, sedangkan **Get Next** atau **Walk** digunakan untuk menelusuri OID berikutnya secara berurutan sehingga seluruh informasi dalam suatu tabel MIB dapat ditampilkan.






