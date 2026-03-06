# 🔗 EtherChannel dan Konfigurasinya

EtherChannel adalah teknologi port aggregation yang dikembangkan oleh Cisco. Intinya, teknologi ini memungkinkan kita untuk menggabungkan beberapa kabel fisik menjadi satu jalur logika tunggal yang jauh lebih cepat dan tangguh.

---

## ❓ Mengapa Kita Butuh EtherChannel?

Tanpa EtherChannel, protokol bernama **Spanning Tree Protocol (STP)** biasanya akan mematikan jalur cadangan untuk mencegah looping. Hasilnya? Kamu punya banyak kabel tapi cuma satu yang terpakai.

Dengan EtherChannel:

- **📈 Bandwidth Bertambah**  
  Kalau kamu menggabungkan 4 kabel @1 Gbps, kamu dapat satu jalur logika sebesar **4 Gbps**.

- **🛡 Redundansi (Backup)**  
  Kalau satu kabel putus, koneksi tidak akan mati. Trafik otomatis pindah ke kabel lain yang masih sehat tanpa perlu menunggu STP menghitung ulang.

- **⚖ Load Balancing**  
  Beban data dibagi rata ke semua kabel yang tergabung dalam grup.

---

# ⚙ Cara Kerja: Protokol Negosiasi

Ada dua cara utama bagaimana perangkat "ngobrol" untuk membentuk EtherChannel:

```
===========================================================================
        PERBANDINGAN PROTOKOL ETHERCHANNEL: LACP vs PAgP
===========================================================================

FITUR           LACP (Link Aggregation)     PAgP (Port Aggregation)
-----------     -----------------------     -----------------------
Standar         IEEE 802.3ad (Terbuka)      Proprietary Cisco
Kecocokan       Semua Vendor Switch         Hanya Perangkat Cisco
Mode            Active & Passive            Desirable & Auto

---------------------------------------------------------------------------
Penjelasan Mode:
- LACP Active   : Memulai negosiasi secara aktif.
- LACP Passive  : Menunggu negosiasi dari lawan (diam).
- PAgP Desirable: Memulai negosiasi secara aktif.
- PAgP Auto     : Menunggu negosiasi dari lawan (diam).
===========================================================================
```

---

# 📜 Aturan Main (Syarat Wajib)

Agar EtherChannel bisa terbentuk, semua kabel/port yang mau digabung harus **"kembar"** dalam hal:

- **⚡ Speed & Duplex**  
  Semuanya harus sama (misal: semua harus **1 Gbps Full Duplex**).

- **🏷 VLAN**  
  Semuanya harus berada di VLAN yang sama atau sama-sama diatur sebagai **Trunk**.

- **🔢 Range**  
  Biasanya maksimal bisa menggabungkan sampai **8 port aktif** dalam satu grup.

---

## 🧪 Topologi Lab

Kali ini saya akan melakukan konfigurasi antar switch pada topologi jaringan di bawah ini.

<img width="623" height="473" alt="topologi EtherChannel" src="https://github.com/user-attachments/assets/2af95082-40d4-448b-9c2c-1826522702bc" />



---

# 🛠 Step-by-step

1. konfigurasi layer 2 etherchannel antara ASW1 dan DSW1 dengan protokol **LACP** dan konfigurasi sebagai **trunk**  
2. konfigurasi layer 2 etherchannel antara ASW2 dan DSW2 dengan protokol **PAgP** dan konfigurasi sebagai **trunk**  
3. konfigurasi **layer 3 eth-channel antara DSW1 dan DSW2 dengan static**  
4. konfigurasi **rute agar pc bisa mencapai SVI**  
5. metode **penyeimbangan beban etherchannel default apa yang digunakan pada setiap switch**  
6. konfigurasi switch untuk **load balancing berdasarkan alamat IP sumber dan tujuan**

---

# 🔧 1. konfigurasi layer 2 etherchannel antara ASW1 dan DSW1 dengan protokol LACP dan konfigurasi sebagai trunk

## 1.a kita konfigurasi ASW1

pertama kita akan lihat **spanning tree** nya dulu untuk melihat port mana yang menjadi root dan yang mana menjadi alternate

```
Interface        Role Sts Cost      Prio.Nbr Type
---------------- ---- --- --------- -------- --------------------------------
Fa0/1            Desg LRN 19        128.1    P2p
Fa0/2            Desg LRN 19        128.2    P2p
Gi0/1            Root FWD 4         128.25   P2p
Gi0/2            Altn BLK 4         128.26   P2p
```

sekarang kita bisa melihat bahwa **G0/1 menjadi root dan G0/2 menjadi alternate**

---

## 1.b konfigurasi EtherChannel g0/1-2

karena pertama menggunakan **LACP** kita bisa memilih menggunakan **active atau passive**  
(jika menggunakan passive pastikan switch lain dalam mode active jika tidak etherchannel tidak akan terbentuk)

namun mari menggunakan **active di kedua sisi** dan jangan lupa menjadikan port ke **trunk**

```
ASW1(config-if-range)#channel-group 1 mode active
ASW1(config-if-range)#int po1
ASW1(config-if)#sw mode trunk
```

<img width="630" height="325" alt="gambar hasil configurasi ASW!" src="https://github.com/user-attachments/assets/59bc094f-4cae-4cf5-ab1a-3118fe4cdb2a" />


kita bisa memastikan dengan perintah

```
sh eth sum
```

<img width="599" height="268" alt="asw1 sh eth sum" src="https://github.com/user-attachments/assets/d537fc55-ec9e-4da9-900e-212019948377" />


lihat, etherchannel sudah dibuat meskipun dalam status **SD (layer 2 down)** dan portnya dengan status **I (standalone)** karena kita belum konfigurasi **DSW1**

---

## 1.c konfigurasi DSW1

lalu kita akan melakukan konfigurasi pada port yang terhubung dengan **ASW1**

```
DSW1(config-if-range)#channel-group 1 mode active
DSW1(config-if-range)#int po1
DSW1(config-if)#sw mode trunk
```

topologi ASW1 dan DSW1 sebelum konfigurasi etherchannel


<img width="623" height="473" alt="sebelum konfigurasi eth ASW1" src="https://github.com/user-attachments/assets/f9597f5b-fc93-4a23-8268-626d00b6b92d" />

sesudah

<img width="637" height="447" alt="sesudah konfigurasi eth ASW1" src="https://github.com/user-attachments/assets/452fec73-8b76-4e92-b2bc-bfc4e77ac2ee" />

lihat sekarang **Spanning Tree sudah tidak menganggap kedua port sebagai port yang berbeda** sehingga tidak perlu memblokir salah satu port untuk menghindari loop.

sekarang kedua port dianggap sebagai **satu aliran besar**

<img width="597" height="266" alt="sh konfigurasi pertama" src="https://github.com/user-attachments/assets/b1579bc7-0a80-4edd-8ab5-275bbfbc2533" />


lihat, sekarang status etherchannel sudah **SU (layer3 in use)** dan port nya **P (in port-channel)** dan protokol menggunakan **LACP**

---

# 🔧 2. konfigurasi layer 2 etherchannel antara ASW2 dan DSW2 dengan protokol PAgP dan konfigurasi sebagai trunk

kita akan melakukan konfigurasi dengan cara yang sama tetapi menggunakan **PAgP**

---

## 2.a konfigurasi ASW2

```
ASW2(config)#int range g0/1-2
ASW2(config-if-range)#channel-group 1 mode desirable
ASW2(config-if-range)#int po1
ASW2(config-if)#sw mode trunk
```

---

## 2.b konfigurasi DSW2

```
DSW2(config)#int range gi1/0/3-4
DSW2(config-if-range)#channel-group 1 mode desirable
DSW2(config-if-range)#int po1
DSW2(config-if)#sw mode trunk
```

kita lihat hasilnya

<img width="593" height="261" alt="sh eth sum DSW2 " src="https://github.com/user-attachments/assets/c978eb9a-4a17-4fd6-8b34-3aaafa284558" />


```
Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----

1      Po1(SU)           PAgP   Gig1/0/3(P) Gig1/0/4(P)
```

kita sudah mengkonfigurasi keduanya dengan benar.

---

# 🔧 3. konfigurasi layer 3 eth-channel antara DSW1 dan DSW2 dengan static

## 3.a konfigurasi DSW2 secara statis

```
DSW2(config-if)#int range g1/0/1-2
DSW2(config-if-range)#no sw
DSW2(config-if-range)#channel-group 2 mode on
Creating a port-channel interface Port-channel 2
DSW2(config-if-range)#int po2
DSW2(config-if)#ip add 10.0.0.2 255.255.255.252
```

---

## 3.b konfigurasi DSW1 secara statis

```
DSW1(config)#int range g1/0/1-2
DSW1(config-if-range)#no sw
DSW1(config-if-range)#channel-group 2 mode on
DSW1(config-if-range)#int po2
DSW1(config-if)#ip add 10.0.0.1 255.255.255.252
```

<img width="593" height="275" alt="sh eth secara statis" src="https://github.com/user-attachments/assets/ac5876aa-c653-4026-8ee9-3b3476ac50d9" />


coba **ping dari DSW1 ke DSW2**

<img width="595" height="117" alt="coba ping" src="https://github.com/user-attachments/assets/a67797f9-0757-43e1-87a4-ba4458956046" />


terlihat bagus.

---

# 🌐 4. konfigurasi rute agar pc bisa mencapai SVI

## 4.a aktifkan routing pada multilayer switch

pada **DSW1**

```
DSW1(config)#ip routing
DSW1(config)#ip route 172.16.2.0 255.255.255.0 10.0.0.2
```

<img width="594" height="251" alt="sh ip route DSW1" src="https://github.com/user-attachments/assets/ab53ebb3-2ef4-49f7-b733-6fbb08e10fab" />


pada **DSW2**

```
DSW2(config)#ip routing
DSW2(config)#ip route 172.16.1.0 255.255.255.0 10.0.0.1
```

<img width="594" height="251" alt="sh ip route DSW2" src="https://github.com/user-attachments/assets/0e15a618-14da-44f2-908a-f7de2528f80a" />


---

## 4.b test koneksi

<img width="595" height="366" alt="tast ping error" src="https://github.com/user-attachments/assets/471d2442-e01b-4f8a-b640-ec2b178ad9e9" />


terlihat **RTO** sehingga perlu dilakukan **troubleshooting**

ternyata masalahnya karena **IP VLAN 1 belum dikonfigurasi pada kedua switch**

```
DSW1(config)#int vlan 1
DSW1(config-if)#ip add 172.16.1.254 255.255.255.0
```

dan pada **DSW2**

```
DSW2(config)#int vlan 1
DSW2(config-if)#ip add 172.16.2.254 255.255.255.0
```

<img width="592" height="218" alt="ping berhasil" src="https://github.com/user-attachments/assets/46f6984e-3767-4021-8f19-ee101406d795" />


akhirnya **ping antar jaringan berhasil**

---

# ⚖ 5. metode penyeimbangan beban etherchannel default

- **ASW1 (C2960)** : src-mac  
- **ASW2 (C2960)** : src-mac  
- **DSW1 (C3650)** : src-mac  
- **DSW2 (C3650)** : src-mac  

---

# ⚙ 6. konfigurasi load balancing etherchannel

mode hashing yang tersedia:

- Source MAC Address  
- Destination MAC Address  
- Source & Destination MAC  
- Source IP / Destination IP  
- Source & Destination Port  

---

## 6.1 ubah load balance pada DSW1

```
DSW1(config)#port-channel load-balance src-dst-ip
DSW1(config)#do sh eth load
```

<img width="496" height="148" alt="load balance DSW1" src="https://github.com/user-attachments/assets/b923cef7-d144-46c2-96fc-72ecfe817b64" />


---

## 6.2 ubah load balance pada DSW2

```
DSW2(config)#port-channel load src-dst-ip
DSW2(config)#do sh eth load
```

<img width="495" height="149" alt="laod balance DSW2" src="https://github.com/user-attachments/assets/189f1c88-8e60-4dc2-b2f9-8586eeb5c5b7" />
