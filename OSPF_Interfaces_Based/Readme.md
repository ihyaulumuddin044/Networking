# OSPF dengan Interface-Based Configuration - Part 2

<img width="905" height="417" alt="topologo ospf part 2" src="https://github.com/user-attachments/assets/b013d7ed-df82-4f3c-ad7c-7b43ebf896c0" />


Pada bagian ini, kita akan melakukan konfigurasi OSPF menggunakan metode **Interface-Based**. Berbeda dengan metode `network` sebelumnya, metode ini jauh lebih presisi karena kita mengaktifkan OSPF langsung di dalam interface terkait.

### Keunggulan Metode Interface-Based:
1. **Presisi:** Menghindari kesalahan hitung *Wildcard Mask*.
2. **Keamanan:** Memastikan OSPF hanya aktif pada interface yang kita inginkan (menghindari kebocoran data ke interface publik).
3. **Keterbacaan:** Lebih mudah dipahami saat melihat konfigurasi melalui perintah `show run`.

---

## 1. Konfigurasi OSPF pada Interface

Kita akan menghapus konfigurasi lama dan menggantinya dengan perintah `ip ospf [process-id] area [area-id]`.

#### --- Konfigurasi pada R1 ---
```
R1(config)# no router ospf 1
R1(config)# interface range g0/0, f1/0, l0
R1(config-if-range)# ip ospf 1 area 0
R1(config-if-range)# exit
R1(config)# router ospf 1
R1(config-router)# passive-interface l0
```
<img width="488" height="273" alt="sh ip pro R1" src="https://github.com/user-attachments/assets/6c1ce7fd-c9ab-4a39-a1e4-20a939922ca3" />

--- Konfigurasi pada R2 & R3 ---


```
--- R2 ---
R2(config)# int range g0/0, f1/0, l0 
R2(config-if-range)# ip ospf 1 area 0
R2(config-if-range)# router ospf 1
R2(config-router)# passive-interface l0

--- R3 ---
R3(config)# int range g0/0, f1/0, f2/0, l0
R3(config-if-range)# ip ospf 1 area 0
R3(config-if-range)# router ospf 1
R3(config-router)# passive-interface l0
```

--- Konfigurasi pada R4 (Metode Passive Default) ---


Di R4, kita menggunakan pendekatan keamanan yang lebih ketat: mematikan semua (passive default) dan hanya membuka interface yang terhubung ke router lain.

```
R4(config)# int range f1/0, f2/0, l0
R4(config-if-range)# ip ospf 1 area 0
R4(config-if-range)# router ospf 1
R4(config-router)# passive-interface default 
R4(config-router)# no passive-interface f1/0 
R4(config-router)# no passive-interface f2/0
```

---


### 2. Optimasi Metrik: Auto-Cost Reference BandwidthOSPF
 OSPF menggunakan Cost sebagai metrik. Rumus standarnya adalah:
 
<img width="363" height="77" alt="rumus ospf bandwitch" src="https://github.com/user-attachments/assets/1a0e466c-13cf-4f1d-8290-2ae6aab90422" />

Secara default, Reference Bandwidth adalah 100 Mbps. Hal ini menimbulkan masalah pada kabel modern:
* FastEthernet (100 Mbps): 100 / 100 = 1 
* GigabitEthernet (1000 Mbps): 100 / 1000 = 0.1 (dibulatkan menjadi 1)
  
Artinya, secara default OSPF tidak bisa membedakan mana yang lebih cepat antara Gigabit dan FastEthernet. 
Agar FastEthernet memiliki cost 100, kita harus mengubah Reference Bandwidth menjadi 10.000 Mbps.

--- Implementasi pada Semua Router ---

Perintah ini wajib dilakukan di seluruh router dalam satu area untuk menghindari routing loop.

`R1, R2, R3, R4(config-router)# auto-cost reference-bandwidth 10000`

Output: % OSPF: Reference bandwidth is changed. Please ensure reference bandwidth is consistent across all routers.

Hasil Verifikasi:
Dengan referensi 10.000 Mbps:
* FastEthernet: 10000 / 100 = 100
* GigabitEthernet: 10000 / 1000 = 10

<img width="678" height="111" alt="bandwitch cost 10000" src="https://github.com/user-attachments/assets/3fcfc32f-884e-4c7f-a03f-6a43518e0836" />

(Gambar: Verifikasi interface cost menunjukkan angka 100 untuk FE)

---

### 3. Konfigurasi R1 sebagai ASBR
Terakhir, kita jadikan R1 sebagai gerbang keluar jaringan (ASBR) dan mengiklankan rute default ke seluruh router OSPF lainnya.

```
R1(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.2
R1(config)# router ospf 1
R1(config-router)# default-information originate
```

** demikian untuk pembahasan ospf part 2 ini, saya akan melanjutkan dengan konfigurasi ospf lainya pada part 3 **



