# 🖧 Konfigurasi VLAN – Part 1 

## 📘 Pengertian VLAN

VLAN (Virtual Local Area Network) adalah teknologi yang memungkinkan satu switch fisik dibagi menjadi beberapa jaringan logis yang terpisah.

Pada praktik ini, VLAN digunakan untuk:

- Memisahkan perangkat berdasarkan kelompok
- Mengurangi broadcast domain
- Meningkatkan keamanan jaringan
- Mengelompokkan user secara logis

---

## 🏷 Jenis VLAN yang Digunakan

Pada praktik ini digunakan:

- VLAN 10
- VLAN 20
- VLAN 30

Setiap VLAN mewakili kelompok perangkat yang berbeda.

---

<img width="987" height="618" alt="gambar topologi Vlan" src="https://github.com/user-attachments/assets/bb5aab1d-b0e0-44da-9354-5b83a413a0d3" />
(gambar topologi)

Pada topologi di atas, setiap PC sudah dikonfigurasi IP address sesuai dengan jaringan VLAN masing-masing.

---

# 1️⃣ Membuat VLAN di Switch

Masuk ke mode konfigurasi:

membuat alamat ip untuk setiap Vlan

<img width="647" height="488" alt="gambar konfigurai ip untuk Vlan" src="https://github.com/user-attachments/assets/708482ea-6f42-4251-b21e-f48a2df8b1dd" />

selanjutnya saya bisa melihat konfigurasi ip yang sudah saya buat dengan:

<img width="645" height="121" alt="gambar show ip int brief router" src="https://github.com/user-attachments/assets/fac29990-371f-41d7-8762-0388fa43b82e" />

---

## 🔹 Membuat VLAN 10

<img width="652" height="230" alt="configurasi Vlan10" src="https://github.com/user-attachments/assets/9c633ec9-cc2e-45ac-888c-dabde4e56ca8" />


---


## 🔹 Membuat VLAN 20

<img width="454" height="61" alt="configurasi Vlan 20" src="https://github.com/user-attachments/assets/a601cdfa-bf9f-4715-a13b-e1d31f42cef2" />


---

## 🔹 Membuat VLAN 30

<img width="454" height="69" alt="configurasi Vlan 30" src="https://github.com/user-attachments/assets/efb1104f-9446-49f9-bdf9-5fd9f4f143dd" />


---

# 2️⃣ Menghubungkan Port ke VLAN (Access Mode)

Port yang terhubung ke PC harus di-set sebagai access mode.

---

### VLAN 10
Port:
- g0/1
- f3/1
- f4/1

---

### VLAN 20
Port:
- g1/1
- f5/1
- f6/1

---

### VLAN 30
Port:
- g2/1
- f7/1
- f8/1

  
---

# 3️⃣ Verifikasi VLAN

Untuk melihat VLAN yang sudah dibuat:


<img width="468" height="176" alt="show konfigurasi vlan" src="https://github.com/user-attachments/assets/f43cc389-b97d-488f-96fc-716910055ba8" />


Pastikan setiap port sudah masuk ke VLAN yang benar.

berikan nama pada Vlan agar mudah dalam proses monitoring

<img width="526" height="264" alt="memberi nama seetiap vlan" src="https://github.com/user-attachments/assets/3f5ac870-aacb-4095-ac7c-17e8042340c5" />

lihat hasilnya:

<img width="468" height="176" alt="show konfigurasi vlan" src="https://github.com/user-attachments/assets/8b066b95-a461-49cc-9107-5b0d8560f768" />


---

# 4️⃣ Pengujian Koneksi

<img width="525" height="441" alt="hasil ping" src="https://github.com/user-attachments/assets/134f0d17-f3dc-4d50-be20-e20f31602c09" />


Lakukan pengujian:

- PC dalam VLAN yang sama → harus bisa saling ping
- PC beda VLAN → masih bisa saling ping karena belum kita berikan kontrol list

Hal ini membuktikan bahwa VLAN berhasil memisahkan broadcast domain.

---

# 🧠 Kesimpulan Part 1

Pada tahap ini kita telah berhasil:

- Membuat VLAN
- Memberikan nama VLAN
- Menghubungkan port ke VLAN
- Menguji isolasi antar VLAN

---

💾 Jangan lupa simpan konfigurasi:

`write` atau `copy run start`

---

✍️ Dokumentasi ini merupakan bagian dari pembelajaran VLAN dasar (segmentation only).



















