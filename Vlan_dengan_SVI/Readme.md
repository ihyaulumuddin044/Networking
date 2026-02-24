# VLAN dengan menggunakan SVI (Switched Virtual Interface)

## 📌 Penjelasan SVI

SVI (Switched Virtual Interface) adalah interface virtual pada switch Layer 3 yang digunakan untuk melakukan routing antar VLAN (inter-VLAN routing).

Berbeda dengan metode **Router-on-a-Stick (RoAS)** yang menggunakan router fisik dengan subinterface, SVI memungkinkan switch Layer 3 melakukan fungsi routing secara langsung tanpa perlu router tambahan.

### Keunggulan SVI

- Proses routing lebih cepat (karena dilakukan langsung oleh switch Layer 3)
- Konfigurasi lebih sederhana dibandingkan RoAS
- Tidak memerlukan trunk ke router untuk inter-VLAN routing internal

Pada praktik ini, saya mengganti metode sebelumnya (RoAS) menjadi metode SVI menggunakan switch Layer 3.

---

## 1. Konfigurasi Router

Kita mulai membuat konfigurasi router terlebih dahulu.  
Sebelumnya saya menggunakan metode RoAS, sehingga konfigurasi lama harus dihapus terlebih dahulu.

<img width="490" height="451" alt="gambar konfigurasi lama" src="https://github.com/user-attachments/assets/b17cf086-1355-4162-8c55-fe715d89dbdf" />


### Cara menghapus konfigurasi VLAN pada router

<img width="581" height="466" alt="gambar hapus configurasi vlan" src="https://github.com/user-attachments/assets/79832372-4e6f-42e5-80b1-b529724290cb" />


Setelah subinterface dihapus, pastikan dengan:

`show ip interface brief` bahwa interface sudah bersih dari konfigurasi VLAN lama.

Kemudian tambahkan IP pada interface g0/0:

`ip add 10.0.0.194 255.255.255.252`

Lakukan pengecekan kembali dengan:

`sh ip int br`

Pastikan interface dalam keadaan up dan IP address sudah sesuai.

---

2. Konfigurasi Switch di SW2 (Switch Layer 3)

Karena SW2 adalah switch Layer 3, maka kita dapat mengaktifkan fungsi routing.

Gunakan perintah:

no switchport

Perintah tersebut berfungsi untuk mengubah port dari Layer 2 menjadi Layer 3 (routed port).

Kemudian berikan alamat IP pada port yang terhubung ke router.

<img width="574" height="136" alt="sh ip int br add ip" src="https://github.com/user-attachments/assets/a100235a-142b-41fa-9557-cf972600f8b7" />


Mengaktifkan routing pada SW2

Agar switch dapat melakukan routing antar VLAN, aktifkan fitur routing:

ip routing

Setelah itu tambahkan default route menuju router:

<img width="552" height="455" alt="Screenshot from 2026-02-24 14-41-58" src="https://github.com/user-attachments/assets/24497db4-2a19-44e2-91a7-03eb46ccdb21" />


Perintah yang digunakan:

`ip route 0.0.0.0 0.0.0.0 10.0.0.194`

Default route ini digunakan agar SW2 dapat meneruskan trafik ke jaringan luar melalui router.

Membuat VLAN pada SW2

Selanjutnya buat VLAN 10 dan 20 pada SW2, lalu masukkan port ke masing-masing VLAN.

<img width="624" height="405" alt="configurasi vlan 10 dan 20 dan 30" src="https://github.com/user-attachments/assets/0fe9d023-7c08-4848-bc52-87161268ebd7" />


Pastikan VLAN berhasil dibuat dengan:

`show vlan brief`

---
Membuat Konfigurasi SVI

Setelah VLAN dibuat, langkah selanjutnya adalah membuat SVI.

```
SW2(config)#int range g1/0/4-5
SW2(config-if-range)#sw mode ac
SW2(config-if-range)#sw ac vlan 10
SW2(config-if-range)#int range g1/0/3
SW2(config-if-range)#sw mode ac
SW2(config-if-range)#sw ac vlan 20
% Access VLAN does not exist. Creating vlan 20
SW2(config-if-range)#vlan 20
SW2(config-vlan)#do sh vlan br

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gig1/0/6, Gig1/0/7, Gig1/0/8, Gig1/0/9
                                                Gig1/0/10, Gig1/0/11, Gig1/0/12, Gig1/0/13
                                                Gig1/0/14, Gig1/0/15, Gig1/0/16, Gig1/0/17
                                                Gig1/0/18, Gig1/0/19, Gig1/0/20, Gig1/0/21
                                                Gig1/0/22, Gig1/0/23, Gig1/0/24, Gig1/1/1
                                                Gig1/1/2, Gig1/1/3, Gig1/1/4
10   VLAN0010                         active    Gig1/0/4, Gig1/0/5
20   VLAN0020                         active    Gig1/0/3
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                    active
```

SVI dibuat dengan masuk ke interface VLAN masing-masing, kemudian memberikan alamat IP.

Contoh:

```
interface vlan 20
ip add 10.0.0.126 255.255.255.192
no shutdown
```
Lakukan konfigurasi yang sama untuk VLAN 10 dan VLAN 30 sesuai dengan IP yang telah ditentukan.

Alamat IP pada SVI ini akan menjadi default gateway untuk setiap VLAN.

Verifikasi Konfigurasi SVI

Gunakan perintah berikut:

sh ip int br
```
GigabitEthernet1/1/3   unassigned      YES unset  down                  down 
GigabitEthernet1/1/4   unassigned      YES unset  down                  down 
Vlan1                  unassigned      YES unset  administratively down down 
Vlan10                 10.0.0.62       YES manual up                    up 
Vlan20                 10.0.0.126      YES manual up                    up 
Vlan30                 10.0.0.190      YES manual up                    up
```



Pastikan:

* Interface VLAN dalam keadaan up

* IP address sudah sesuai

* ip routing sudah aktif

* Jika interface VLAN dalam keadaan down, pastikan ada minimal satu port aktif dalam VLAN tersebut.

---

4. Tes Koneksi

Setelah konfigurasi selesai, lakukan pengujian koneksi menggunakan perintah:

`ping`

<img width="799" height="595" alt="koneksi" src="https://github.com/user-attachments/assets/6cb26d25-44a7-4a06-833b-c2d4e7f777bf" />


Dari hasil pengujian terlihat bahwa:

* Koneksi antar jaringan sudah dapat dilakukan

* Terdapat request time out pada awal ping (hal ini normal karena proses ARP)

* Setelah itu koneksi berjalan dengan baik

Pada tahap ini, switch Layer 3 telah berhasil melakukan routing antar VLAN menggunakan metode SVI tanpa menggunakan Router-on-a-Stick.

📌 Kesimpulan

* Pada praktik ini saya telah:

* Menghapus konfigurasi RoAS pada router

* Mengaktifkan routing pada switch Layer 3

* Membuat VLAN 10, 20, dan 30

* Membuat SVI sebagai gateway masing-masing VLAN

* Menambahkan default route menuju router

* Melakukan pengujian koneksi antar jaringan

Metode SVI lebih efisien dibandingkan RoAS karena proses routing dilakukan langsung oleh switch Layer 3.








