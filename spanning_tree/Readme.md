# Konfigurasi STP (PVST+)

PVST+ adalah protokol jaringan milik Cisco yang merupakan pengembangan dari STP (Spanning Tree Protocol) standar.

- **STP Standar (IEEE 802.1D):** Hanya menjalankan satu instansi spanning-tree untuk seluruh jaringan, tanpa memperhatikan jumlah VLAN.
- **PVST+:** Memungkinkan switch menjalankan instansi STP terpisah untuk setiap VLAN.

## Mengapa PVST+ Penting?

Tanpa PVST+, jika ada jalur redundant di jaringan, STP standar akan memblokir satu jalur tersebut untuk semua VLAN.

Dengan PVST+:

- **Load Balancing:** Jalur A bisa aktif untuk VLAN 10, sementara Jalur B aktif untuk VLAN 20.
- **Fleksibilitas:** Gangguan pada satu VLAN tidak memengaruhi VLAN lain.
- **Interoperabilitas:** Tanda "+" menunjukkan kompatibilitas dengan STP standar (IEEE).

---

## Topologi STP

<img width="752" height="397" alt="topologi STP" src="https://github.com/user-attachments/assets/c03fd919-f58e-48a4-94be-a796b9dbe7f3" />


Pada topologi ini digunakan 4 switch dan 2 PC dengan subnet berbeda.

---

# 1. Identifikasi Root Bridge

Langkah pertama adalah mengecek root bridge secara default.

### Cek pada SW1

```
SW1>en
SW1#sh span
```
<img width="540" height="259" alt="sh span sw1 p1" src="https://github.com/user-attachments/assets/d4a741bc-c305-4a35-b1e4-36fdfac0b38a" />


Output menunjukkan:

```
Root ID    Priority    32769
Address     0000.0C85.3B12
Port        3(FastEthernet0/3)
```

Artinya SW1 **bukan root**, karena root berada di port Fa0/3 (terhubung ke SW2).

---

### Cek pada SW2

```
SW2(config)#do sh span
```
<img width="536" height="277" alt="sh span SW2" src="https://github.com/user-attachments/assets/38ac28e7-d959-4bc8-893b-b452b81f8c9b" />

Output menunjukkan:

```
This bridge is the root
```

SW2 menjadi root secara default karena memiliki MAC address paling kecil.


---

# 2. Mengatur Root Bridge

Kita ingin menjadikan:

- SW1 sebagai **Root Primary VLAN 1**
- SW2 sebagai **Root Primary VLAN 2**
- Masing-masing menjadi secondary untuk VLAN lainnya

---

## Konfigurasi pada SW1

```
SW1#conf t
SW1(config)#spanning-tree vlan 1 root primary
SW1(config)#spanning-tree vlan 2 root secondary
```

---

## Konfigurasi pada SW2

```
SW2#conf t
SW2(config)#spanning-tree vlan 1 root secondary
SW2(config)#spanning-tree vlan 2 root primary
```

---

## Verifikasi pada SW1

```
SW1#sh span
```

Output menunjukkan:

```
This bridge is the root
```

SW1 sekarang menjadi root untuk VLAN 1.

Namun VLAN 2 belum muncul karena belum dibuat.

---

# Membuat VLAN 2 dan Konfigurasi Trunk

```
SW1(config)#vlan 2
SW1(config)#int range f0/1-3
SW1(config-if-range)#switchport mode trunk
SW1(config-if-range)#no shutdown
SW1(config)#do sh span
```


Sekarang VLAN 2 sudah muncul dalam output `show spanning-tree`.

Lakukan konfigurasi VLAN dan trunk yang sama pada SW2.
gambar topoligi setelah pengantian root:

<img width="836" height="384" alt="topologi setelah pengantian root" src="https://github.com/user-attachments/assets/8fb769b5-3def-4b05-a170-8f694ee3a719" />

---



# 3. Konfigurasi SW3 dan SW4

Port yang terhubung ke end device (PC) perlu diaktifkan PortFast agar tidak menunggu proses STP convergence.

⚠️ Jangan aktifkan PortFast pada port trunk!

---

## Mengaktifkan PortFast

```
SW3(config)#int range fa0/3-4
SW3(config-if-range)#spanning-tree portfast
```

---

## Mengaktifkan BPDU Guard

Untuk mencegah switch baru terhubung dan mengganggu topologi:

```
SW3(config-if-range)#spanning-tree bpduguard enable
```

Jika ada switch baru terhubung, port akan otomatis masuk status err-disable:

```
%SPANTREE-2-BLOCK_BPDUGUARD
%PM-4-ERR_DISABLE
```

---

# 4. Pengujian Konfigurasi

<img width="795" height="520" alt="SW baru terblokir" src="https://github.com/user-attachments/assets/9a774286-3979-482f-b463-fcbba0d282ff" />


Hasil:

- Switch baru otomatis terblokir karena BPDU Guard aktif.
- PC tetap dapat berkomunikasi normal.
- Topologi tetap stabil.

---

# Kesimpulan

Dengan PVST+:

- Kita bisa menentukan root berbeda untuk tiap VLAN.
- Dapat melakukan load balancing antar jalur.
- PortFast mempercepat koneksi end device.
- BPDU Guard meningkatkan keamanan topologi.

Konfigurasi STP yang tepat membuat jaringan lebih stabil, aman, dan efisien.
