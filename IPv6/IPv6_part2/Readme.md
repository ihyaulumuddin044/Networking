# Panduan IPv6 Part 2: Konfigurasi EUI-64, Default Gateway, dan Statis Routing

Dokumentasi ini membahas implementasi tingkat lanjut IPv6, meliputi otomatisasi pengalamatan menggunakan **EUI-64 (SLAAC)**, pemosisian *Default Gateway*, serta konfigurasi **IPv6 Static Route** menggunakan *Link-Local Address*.

---

<img width="639" height="252" alt="topologi jaringan" src="https://github.com/user-attachments/assets/e439f47c-0cb5-4595-b523-f1306992e5d7" />

## 1. Addressing Table

| Device | Interface | IPv6 Address / Prefix | Link-Local Address | Default Gateway |
| :--- | :--- | :--- | :--- | :--- |
| **R1** | G0/0 | *Unassigned (Link-Local Only)* | `FE80::206:2AFF:FE56:5B01` | N/A |
| | G0/1 | `2001:DB8::206:2AFF:FE56:5B02/64` *(EUI-64)* | `FE80::206:2AFF:FE56:5B02` | N/A |
| **R2** | G0/0 | *Unassigned (Link-Local Only)* | `FE80::2D0:FFFF:FE63:C78B` | N/A |
| | G0/1 | `2001:DB8:0:1:207:ECFF:FE9A:7C53/64` *(EUI-64)* | `FE80::207:ECFF:FE9A:7C53` | N/A |
| **PC0** | NIC | `2001:DB8::2/64` | `FE80::201:43FF:FE0B:EBC9` | `2001:DB8::206:2AFF:FE56:5B02` |
| **PC1** | NIC | `2001:DB8:0:1::2/64` | `FE80::250:FFF:FE05:7054` | `2001:DB8:0:1:207:ECFF:FE9A:7C53` |

---

## 2. Konfigurasi EUI-64 (SLAAC)

**SLAAC (Stateless Address Autoconfiguration)** adalah mekanisme yang memungkinkan *host* atau *interface* router mengonfigurasi alamat IPv6 unik mereka secara otomatis tanpa server DHCPv6. Salah satu metodenya adalah **EUI-64**, yang memanfaatkan MAC Address perangkat untuk membuat 64-bit Interface ID.



> **Catatan:** Untuk melihat MAC Address pada router Cisco, gunakan perintah `show interfaces [nama_interface]`.

<img width="567" height="226" alt="melihat alamat mac int" src="https://github.com/user-attachments/assets/9a2c7f33-d5c7-4789-9f48-60f84728e921" />

### Konfigurasi pada R1
Aktifkan IPv6 routing terlebih dahulu, kemudian gunakan parameter `eui-64` di akhir perintah pengalamatan:
```
R1(config)# ipv6 unicast-routing 
R1(config)# interface g0/1
R1(config-if)# ipv6 address 2001:db8::/64 eui-64
```
Verifikasi Hasil R1:

` R1(config-if)# do show ipv6 interface brief `

Output:

GigabitEthernet0/1 [up/up]: FE80::206:2AFF:FE56:5B02 | 2001:DB8::206:2AFF:FE56:5B02

Konfigurasi pada R2
```
R2(config)# ipv6 unicast-routing 
R2(config)# interface g0/1
R2(config-if)# ipv6 address 2001:db8:0:1::/64 eui-64
Verifikasi Hasil R2:
R2(config-if)# do show ipv6 interface brief
```
Output:

GigabitEthernet0/1 [up/up]: FE80::207:ECFF:FE9A:7C53 | 2001:DB8:0:1:207:ECFF:FE9A:7C53

(Penting: Hasil akhir segmen host EUI-64 pada lab Anda akan berbeda tergantung MAC Address fisik perangkat).


## 3. Aktivasi Link-Local pada Interface Antar-Router
Untuk menghubungkan R1 G0/0 dan R2 G0/0, kita cukup mengaktifkan IPv6 pada interface tersebut agar mendapatkan alamat Link-Local otomatis tanpa perlu memberikan alamat IPv6 global eksplisit.

Pada R1:
```
R1(config)# interface g0/0
R1(config-if)# ipv6 enable
```

Pada R2:
```
R2(config)# interface g0/0
R2(config-if)# ipv6 enable
```

Setelah diaktifkan, kedua interface akan otomatis menghasilkan alamat berawalan FE80:: yang akan kita gunakan sebagai poin Next-Hop pada langkah routing statis.

## 4. Konfigurasi IPv6 Static Route & Troubleshooting
Konfigurasi rute statis pada IPv6 memiliki logika yang sama dengan IPv4. Namun, ada aturan ketat jika Anda memilih menggunakan Link-Local Address sebagai Next-Hop.

Kasus Error pada R1:
Saat mencoba memasukkan rute statis biasa seperti ini:

```
R1(config)# ipv6 route 2001:db8:0:1::/64 FE80::2D0:FFFF:FE63:C78B
Router akan menolak dengan memunculkan pesan error:
```

% Interface has to be specified for a link-local nexthop

Mengapa ini terjadi?
Alamat Link-Local (FE80::) tidak bersifat global dan bisa saja sama di beberapa interface router. Oleh karena itu, router meminta kejelasan lewat interface mana ia harus melompat ke alamat tersebut.

Solusi Perintah yang Benar:
Sebutkan interface exit lokal (g0/0) sebelum menuliskan alamat Link-Local Next-Hop.

Konfigurasi R1 (Menuju Network PC1):
```
R1(config)# ipv6 route 2001:db8:0:1::/64 g0/0 FE80::2D0:FFFF:FE63:C78B
```
Konfigurasi R2 (Menuju Network PC0):

` R2(config)# ipv6 route 2001:db8::/64 g0/0 FE80::206:2AFF:FE56:5B01 `

## 5. Uji Konektivitas (Ping)
Setelah melakukan konfigurasi IP secara statis pada PC0 dan PC1 (mengacu pada Addressing Table), lakukan pengujian end-to-end ping dari PC0 menuju alamat IPv6 PC1 (2001:db8:0:1::2).

` C:\> ping 2001:db8:0:1::2 `

<img width="495" height="375" alt="ping pc0 ke pc1" src="https://github.com/user-attachments/assets/5aa11245-5703-452b-afd9-611397b8b823" />

Jika konfigurasi routing statis dengan Link-Local berjalan dengan benar, paket data akan mendapat respons Reply dari PC seberang, menandakan jaringan telah terhubung sempurna.
