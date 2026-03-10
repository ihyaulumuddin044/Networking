# OSPF dan Konfigurasinya - Part 1

**Open Shortest Path First (OSPF)** adalah protokol routing dinamis berbasis *Link-State* yang digunakan untuk mendistribusikan informasi routing dalam satu sistem otonom (*Autonomous System/AS*). Dikembangkan oleh IETF, OSPF merupakan standar terbuka yang memungkinkan perangkat dari berbagai vendor saling berkomunikasi.

### Karakteristik Utama
* **Algoritma Dijkstra:** OSPF menggunakan algoritma *Shortest Path First* (SPF) untuk menghitung jalur terpendek berdasarkan total biaya (Cost) terendah.
* **Metrik Cost:** Jalur dipilih berdasarkan bandwidth; semakin besar bandwidth, semakin rendah biayanya.
* **Konvergensi Cepat:** Mampu mendeteksi perubahan topologi dengan cepat dan segera menghitung ulang rute baru.
* **Efisiensi Bandwidth:** Update hanya dikirimkan melalui *Link-State Advertisements* (LSA) saat terjadi perubahan jaringan (tidak mengirim seluruh tabel routing terus-menerus).

---
<img width="1043" height="528" alt="topologi OSPF" src="https://github.com/user-attachments/assets/a48b7075-91e5-49c1-8c5d-8a45e7ec0a45" />


Kita akan mencoba mengonfigurasi OSPF pada topologi di atas. Panduan ini tidak akan menjelaskan cara memberikan IP Address pada interface fisik, pastikan Anda sudah mengaturnya terlebih dahulu.

---

### 1. Konfigurasi Loopback pada Setiap Router
Interface Loopback digunakan sebagai identitas router (Router ID) yang stabil. Berikut adalah daftar IP yang digunakan:

| Router | IP Address | Netmask | Interface |
| :--- | :--- | :--- | :--- |
| R1 | 1.1.1.1 | 255.255.255.255 | Loopback0 |
| R2 | 2.2.2.2 | 255.255.255.255 | Loopback0 |
| R3 | 3.3.3.3 | 255.255.255.255 | Loopback0 |
| R4 | 4.4.4.4 | 255.255.255.255 | Loopback0 |

**Command Configuration (CLI):**

```
--- KONFIGURASI R1 ---
R1(config)# interface loopback 0
R1(config-if)# ip address 1.1.1.1 255.255.255.255

--- KONFIGURASI R2 ---
R2(config)# interface loopback 0
R2(config-if)# ip address 2.2.2.2 255.255.255.255

--- KONFIGURASI R3 ---
R3(config)# interface loopback 0
R3(config-if)# ip address 3.3.3.3 255.255.255.255


--- KONFIGURASI R4 ---
R4(config)# interface loopback 0
R4(config-if)# ip address 4.4.4.4 255.255.255.255
```

---

### 2. Konfigurasi OSPF dan Interface
Aktifkan OSPF pada setiap router. Gunakan Wildcard Mask (kebalikan dari netmask) untuk menentukan network yang didaftarkan ke dalam Area 0.
Catatan: Jangan aktifkan OSPF pada link internet di R1. Gunakan perintah passive-interface pada interface yang tidak perlu mengirim paket
hello (seperti loopback atau ke arah user) untuk meningkatkan keamanan dan efisiensi.


 --- Konfigurasi pada R4 ---
```
R4(config)# router ospf 4
R4(config-router)# network 192.168.4.0 0.0.0.255 area 0
R4(config-router)# network 10.0.24.0 0.0.0.3 area 0
R4(config-router)# network 10.0.34.0 0.0.0.3 area 0
R4(config-router)# passive-interface g0/0
R4(config-router)# passive-interface loopback 0
```

<img width="478" height="283" alt="hasil sopf r4" src="https://github.com/user-attachments/assets/0d579c9f-b969-48c9-96ee-30c115912d0e" />

 --- Konfigurasi pada R3 ---

```
R3(config)# router ospf 3
R3(config-router)# network 10.0.13.0 0.0.0.3 area 0
R3(config-router)# network 10.0.34.0 0.0.0.3 area 0
R3(config-router)# network 3.3.3.3 0.0.0.0 area 0
R3(config-router)# passive-interface loopback 0
```

--- Konfigurasi pada R2 ---

```
R2(config)# router ospf 2
R2(config-router)# network 10.0.0.0 0.0.255.255 area 0
R2(config-router)# network 2.2.2.2 0.0.0.0 area 0
R2(config-router)# passive-interface loopback 0
```

--- Konfigurasi pada R1 ---

R1 bertindak sebagai penghubung ke ISP. Kita gunakan perintah `default-information 
originate` agar R1 menyebarkan rute default ke router lain dalam area OSPF, menjadikannya
sebagai ASBR (Autonomous System Boundary Router).

```
R1(config)# router ospf 1
R1(config-router)# network 10.0.12.0 0.0.0.3 area 0
R1(config-router)# network 10.0.13.0 0.0.0.3 area 0
R1(config-router)# network 1.1.1.1 0.0.0.0 area 0
R1(config-router)# passive-interface loopback 0
R1(config-router)# default-information originate
```

<img width="501" height="313" alt="R1 sebagai ASBR" src="https://github.com/user-attachments/assets/28840370-d5a3-4821-a5f9-02ea8be6a9a4" />

---

### 3. Cek Tabel Routing di R2, R3, dan R4
Gunakan perintah show ip route untuk memverifikasi apakah rute sudah dipelajari.

* C: Terhubung langsung (Connected).

* O: Rute yang dipelajari melalui OSPF.

* O*E2: Rute default eksternal yang berasal dari R1 (ASBR).

=== Pada R2 ===


<img width="553" height="338" alt="ip route R2" src="https://github.com/user-attachments/assets/c76eef5d-dc44-4b40-8429-cde7381593db" />

Hasil yang diharapkan: Rute default tersedia via R1.
O*E2 0.0.0.0/0 [110/1] via 10.0.12.1, GigabitEthernet0/0

=== Pada R3 ===


<img width="560" height="341" alt="Ip route R3" src="https://github.com/user-attachments/assets/b7338c11-cece-477d-ad08-3ef6b6a69257" />


O*E2 0.0.0.0/0 [110/1] via 10.0.13.1, FastEthernet1/0

=== Pada R4 ===


<img width="585" height="376" alt="ip route R4" src="https://github.com/user-attachments/assets/aeb50b8b-1e09-4b95-8ae3-314e3f04f3ea" />


Pada R4, terjadi Equal Cost Multi-Path (ECMP) atau load balance, di mana rute ke luar memiliki dua jalur dengan cost yang sama.








