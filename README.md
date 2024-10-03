# jarkom-modul-2-IT42-2024

## Topologi DNS Master/Slave dan Konfigurasi Web Server
![image](https://github.com/user-attachments/assets/4e2bb1b1-7b9c-49c1-8c89-5a991dd2afd6)

## Persiapan Awal (Prerequisite)

### DNS MASTER/SLAVE
Jalankan perintah berikut untuk memperbarui dan menginstal `bind9` pada DNS Master dan Slave:
```bash
apt-get update
apt-get install bind9 -y
```

### CLIENT
Untuk mengonfigurasi DNS pada klien, tambahkan alamat nameserver ke file `resolv.conf`:
```bash
echo nameserver 10.84.2.3 >> /etc/resolv.conf
```

### Router
Tambahkan konfigurasi nameserver pada router:
```bash
echo nameserver 192.168.122.1 >> /etc/resolv.conf
```

---

## Soal 1
Untuk mempersiapkan peperangan World War MMXXIV (Iya sebanyak itu), Sriwijaya membuat dua kotanya menjadi web server yaitu Tanjungkulai, dan Bedahulu, serta Sriwijaya sendiri akan menjadi DNS Master. Kemudian karena merasa terdesak, Majapahit memberikan bantuan dan menjadikan kerajaannya (Majapahit) menjadi DNS Slave.

### Nusantara (Router)
```bash
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
  address 10.84.1.1
  netmask 255.255.255.0

auto eth2
iface eth2 inet static
  address 10.84.2.1
  netmask 255.255.255.0

auto eth3
iface eth3 inet static
  address 10.84.3.1
  netmask 255.255.255.0

up iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.84.0.0/16
```

### Sriwijaya (DNS Master)
```bash
auto eth0
iface eth0 inet static
	address 10.84.2.3
	netmask 255.255.255.0
	gateway 10.84.2.1

up echo nameserver 192.168.122.1 > /etc/resolv.conf
```

### Majapahit (DNS Slave)
```bash
auto eth0
iface eth0 inet static
	address 10.84.1.2
	netmask 255.255.255.0
	gateway 10.84.1.1
	
up echo nameserver 192.168.122.1 > /etc/resolv.conf
```

### Sanjaya (Client)
```bash
auto eth0
iface eth0 inet static
	address 10.84.2.2
	netmask 255.255.255.0
	gateway 10.84.2.1

up echo nameserver 10.84.2.3 >> /etc/resolv.conf
```

### Jayanegara (Client)
```bash
auto eth0
iface eth0 inet static
	address 10.84.1.5
	netmask 255.255.255.0
	gateway 10.84.1.1
	
up echo nameserver 10.84.2.3 >> /etc/resolv.conf
```

### Anusapati (Client)
```bash
auto eth0
iface eth0 inet static
	address 10.84.1.3
	netmask 255.255.255.0
	gateway 10.84.1.1
	
up echo nameserver 10.84.2.3 >> /etc/resolv.conf
```

### Tanjungkulai (Web Server)
```bash
auto eth0
iface eth0 inet static
	address 10.84.2.4
	netmask 255.255.255.0
	gateway 10.84.2.1
```

### Bedahulu (Web Server)
```bash
auto eth0
iface eth0 inet static
	address 10.84.2.5
	netmask 255.255.255.0
	gateway 10.84.2.1
```

### Kotalingga (Web Server)
```bash
auto eth0
iface eth0 inet static
	address 10.84.1.4
	netmask 255.255.255.0
	gateway 10.84.1.1
```

### Solok (Load Balancer)
```bash
auto eth0
iface eth0 inet static
	address 10.84.3.2
	netmask 255.255.255.0
	gateway 10.84.3.1
```

---

## Soal 2
Karena para pasukan membutuhkan koordinasi untuk melancarkan serangannya, maka buatlah sebuah domain yang mengarah ke Solok dengan alamat sudarsana.xxxx.com dengan alias www.sudarsana.xxxx.com, dimana xxxx merupakan kode kelompok. Contoh: sudarsana.it01.com.

### Domain untuk Load Balancer: `sudarsana.it42.com`
1. Buat zona baru di Sriwijaya (DNS Master):
   ```bash
   nano /etc/bind/named.conf.local
   ```
   Tambahkan konfigurasi berikut:
   ```
   zone "sudarsana.it42.com" {
       type master;
       file "/etc/bind/it42/sudarsana.it42.com";
   };
   ```

2. Salin template `db.local` untuk zona baru:
   ```bash
   cp /etc/bind/db.local /etc/bind/it42/sudarsana.it42.com
   ```

3. Edit file `sudarsana.it42.com`:
   ```bash
   nano /etc/bind/it42/sudarsana.it42.com
   ```
   Isi file sebagai berikut:
   ```
   $TTL    604800
   @       IN      SOA     sudarsana.it42.com. root.sudarsana.it42.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
   ;
   @       IN      NS      sudarsana.it42.com.
   @       IN      A       10.84.3.1
   www     IN      CNAME   sudarsana.it42.com.
   ```

4. Restart layanan `bind9`:
   ```bash
   service bind9 restart
   ```

### Soal 3
Para pasukan juga perlu mengetahui mana titik yang akan diserang, sehingga dibutuhkan domain lain yaitu pasopati.xxxx.com dengan alias www.pasopati.xxxx.com yang mengarah ke Kotalingga.

1. Tambahkan zona baru di Sriwijaya:
   ```bash
   nano /etc/bind/named.conf.local
   ```
   Tambahkan konfigurasi berikut:
   ```
   zone "pasopati.it42.com" {
       type master;
       file "/etc/bind/it42/pasopati.it42.com";
   };
   ```

2. Salin file `db.local` untuk zona baru:
   ```bash
   cp /etc/bind/db.local /etc/bind/it42/pasopati.it42.com
   ```

3. Edit file `pasopati.it42.com`:
   ```bash
   nano /etc/bind/it42/pasopati.it42.com
   ```
   Isi file sebagai berikut:
   ```
   $TTL    604800
   @       IN      SOA     pasopati.it42.com. root.pasopati.it42.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
   ;
   @       IN      NS      pasopati.it42.com.
   @       IN      A       10.84.1.4
   www     IN      CNAME   pasopati.it42.com.
   ```

4. Restart layanan `bind9`:
   ```bash
   service bind9 restart
   ```

### Soal 4
Markas pusat meminta dibuatnya domain khusus untuk menaruh informasi persenjataan dan suplai yang tersebar. Informasi dan suplai meme terbaru tersebut mengarah ke Tanjungkulai dan domain yang ingin digunakan adalah rujapala.xxxx.com dengan alias www.rujapala.xxxx.com.

1. Tambahkan zona baru di Sriwijaya:
   ```bash
   nano /etc/bind/named.conf.local
   ```
   Tambahkan konfigurasi berikut:
   ```
   zone "rujapala.it42.com" {
       type master;
       file "/etc/bind/it42/rujapala.it42.com";
   };
   ```

2. Salin file `db.local` untuk zona baru:
   ```bash
   cp /etc/bind/db.local /etc/bind/it42/rujapala.it42.com
   ```

3. Edit file `rujapala.it42.com`:
   ```bash
   nano /etc/bind/it42/rujapala.it42.com
   ```
   Isi file sebagai berikut:
   ```
   $TTL    604800
   @       IN      SOA     rujapala.it42.com. root.rujapala.it42.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
   ;
   @       IN      NS      rujapala.it42.com.
   @       IN      A       10.84.2.4
   www     IN      CNAME   rujapala.it42.com.
   ```

4. Restart layanan `bind9`:
   ```bash
   service bind9 restart
   ```
### Soal 5
Pastikan semua client dapat mengakses ke domain-domain tersebut
![image](https://github.com/user-attachments/assets/57304f2b-6e86-41e6-b2e8-d26c84e9ed50)



### soal 6
Beberapa daerah memiliki keterbatasan yang menyebabkan hanya dapat mengakses domain secara langsung melalui alamat IP domain tersebut. Karena daerah tersebut tidak diketahui secara spesifik, pastikan semua komputer (client) dapat mengakses domain pasopati.xxxx.com melalui alamat IP Kotalingga (Notes: menggunakan pointer record).


1. **Menambahkan Zona Reverse DNS di DNS Master (Sriwijaya)**  
   Untuk memastikan bahwa alamat IP dapat diterjemahkan kembali menjadi nama domain, pertama-tama dibuat zona reverse DNS. Zona reverse ini memungkinkan perangkat dalam jaringan untuk melakukan query terbalik dari IP ke nama host.

   Konfigurasi dilakukan dengan menambahkan zona berikut pada file **named.conf.local** di DNS Master (Sriwijaya):
   ```bash
   zone "1.84.10.in-addr.arpa" {
       type master;
       file "/etc/bind/it42/1.84.10.in-addr.arpa";
   };
   ```

2. **Membuat File Zona PTR**  
   Setelah menambahkan zona reverse, dibuat file konfigurasi PTR yang memetakan IP **10.84.1.4** ke domain **pasopati.it42.com**. Konfigurasi PTR ini diisi sebagai berikut:
   ```bash
   $TTL    604800
   @       IN      SOA     pasopati.it42.com. root.pasopati.it42.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
   ;
   @       IN      NS      pasopati.it42.com.
   4       IN      PTR     pasopati.it42.com.
   ```

   Pada konfigurasi ini, angka **4** pada record PTR mengacu pada oktet terakhir dari IP **10.84.1.4**, yang kemudian dipetakan ke domain **pasopati.it42.com**.

3. **Restart Layanan BIND**  
   Untuk menerapkan perubahan yang dilakukan, layanan **bind9** direstart dengan perintah:
   ```bash
   service bind9 restart
   ```

4. **Pengujian Reverse DNS**  
   Setelah konfigurasi PTR selesai, dilakukan pengujian untuk memastikan bahwa reverse DNS lookup berfungsi. Pengujian dilakukan menggunakan perintah **dig** sebagai berikut:
   ```bash
   dig -x 10.84.1.4
   ```
   Hasil yang diharapkan adalah resolusi IP **10.84.1.4** menjadi domain **pasopati.it42.com**.

---


### Soal 7
Akhir-akhir ini seringkali terjadi serangan brainrot ke DNS Server Utama, sebagai tindakan antisipasi kamu diperintahkan untuk membuat DNS Slave di Majapahit untuk semua domain yang sudah dibuat sebelumnya yang mengarah ke Sriwijaya.

1. Buka file konfigurasi BIND di Majapahit:
   ```bash
   nano /etc/bind/named.conf.local
   ```

2. Tambahkan konfigurasi untuk setiap domain yang ada di DNS Master (Sriwijaya):
   - **Domain sudarsana.it42.com**:
     ```bash
     zone "sudarsana.it42.com" {
         type slave;
         masters { 10.84.2.3; };  # IP Sriwijaya (DNS Master)
         file "/var/cache/bind/sudarsana.it42.com";
     };
     ```

   - **Domain pasopati.it42.com**:
     ```bash
     zone "pasopati.it42.com" {
         type slave;
         masters { 10.84.2.3; };  # IP Sriwijaya (DNS Master)
         file "/var/cache/bind/pasopati.it42.com";
     };
     ```

   - **Domain rujapala.it42.com**:
     ```bash
     zone "rujapala.it42.com" {
         type slave;
         masters { 10.84.2.3; };  # IP Sriwijaya (DNS Master)
         file "/var/cache/bind/rujapala.it42.com";
     };
     ```

Pada konfigurasi ini, **type slave** menunjukkan bahwa Majapahit bertindak sebagai DNS Slave, dan **masters { 10.84.2.3; }** mengidentifikasi **Sriwijaya** (DNS Master) yang memegang zona master dari domain tersebut. Setiap zona disimpan di **/var/cache/bind/** pada Majapahit,selanjutnya DNS Master (Sriwijaya) perlu mengizinkan DNS Slave (Majapahit) untuk menyalin zona dari server. Hal ini dilakukan dengan mengatur transfer zona di **Sriwijaya**.

1. Buka file **named.conf.local** di Sriwijaya:
   ```bash
   nano /etc/bind/named.conf.local
   ```

2. Tambahkan pengaturan transfer zona untuk setiap domain, yang mengizinkan transfer ke IP **Majapahit** (10.84.1.2):
   - **Untuk sudarsana.it42.com**:
     ```bash
     zone "sudarsana.it42.com" {
         type master;
         file "/etc/bind/it42/sudarsana.it42.com";
         allow-transfer { 10.84.1.2; };  # Majapahit (DNS Slave)
     };
     ```

   - **Untuk pasopati.it42.com**:
     ```bash
     zone "pasopati.it42.com" {
         type master;
         file "/etc/bind/it42/pasopati.it42.com";
         allow-transfer { 10.84.1.2; };  # Majapahit (DNS Slave)
     };
     ```

   - **Untuk rujapala.it42.com**:
     ```bash
     zone "rujapala.it42.com" {
         type master;
         file "/etc/bind/it42/rujapala.it42.com";
         allow-transfer { 10.84.1.2; };  # Majapahit (DNS Slave)
     };
     ```

Konfigurasi ini memungkinkan **Majapahit** untuk menyalin zona secara otomatis dari **Sriwijaya** ketika terjadi perubahan atau pembaruan di zona tersebut,
Setelah melakukan perubahan pada konfigurasi DNS di Majapahit dan Sriwijaya, langkah selanjutnya adalah me-restart layanan **bind9** pada kedua server untuk menerapkan perubahan.

- **Di Majapahit (DNS Slave)**:
  ```bash
  service bind9 restart
  ```

- **Di Sriwijaya (DNS Master)**:
  ```bash
  service bind9 restart
  ```


Setelah konfigurasi dan restart, perlu dilakukan verifikasi apakah DNS Slave di Majapahit sudah berhasil mereplikasi zona dari DNS Master (Sriwijaya). Langkah verifikasi dilakukan dengan memeriksa cache BIND di Majapahit untuk memastikan file zona tersedia:

1. Periksa direktori **/var/cache/bind** di Majapahit:
   ```bash
   ls /var/cache/bind/
   ```

2. Pastikan terdapat file zona yang sesuai, seperti:
   - `sudarsana.it42.com`
   - `pasopati.it42.com`
   - `rujapala.it42.com`

Jika file-file ini ada, artinya replikasi zona berhasil.

---

## No 8
Kamu juga diperintahkan untuk membuat subdomain khusus melacak kekuatan tersembunyi di Ohio dengan subdomain cakra.sudarsana.xxxx.com yang mengarah ke Bedahulu.
### Script Konfigurasi
```bash
#!/bin/bash
echo 'zone "sudarsana.it42.com" {
    type master;
    notify yes;
    file "/etc/bind/jarkom/sudarsana.it42.com";
};' > /etc/bind/named.conf.local

cp /etc/bind/db.local /etc/bind/jarkom/sudarsana.it42.com

echo '
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     sudarsana.it42.com. root.sudarsana.it42.com. (
                        2024100201      ; Serial
                        604800         ; Refresh
                        86400         ; Retry
                        2419200         ; Expire
                        604800 )       ; Negative Cache TTL
;
@       IN      NS      sudarsana.it42.com.
@       IN      A       10.84.3.1     ; IP Solok
www     IN      CNAME   sudarsana.it42.com.
cakra	IN      A       10.84.2.5     ; IP Bedahulu' > /etc/bind/jarkom/sudarsana.it42.com

service bind9 restart
```

## No 9
Karena terjadi serangan DDOS oleh shikanoko nokonoko koshitantan (NUN), sehingga sistem komunikasinya terhalang. Untuk melindungi warga, kita diperlukan untuk membuat sistem peringatan dari siren man oleh Frekuensi Freak dan memasukkannya ke subdomain panah.pasopati.xxxx.com dalam folder panah dan pastikan dapat diakses secara mudah dengan menambahkan alias www.panah.pasopati.xxxx.com dan mendelegasikan subdomain tersebut ke Majapahit dengan alamat IP menuju radar di Kotalingga.

### Script Konfigurasi
```bash
echo '
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     pasopati.it42.com. root.pasopati.it42.com. (
                        2024100201      ; Serial
                        604800         ; Refresh
                        86400         ; Retry
                        2419200         ; Expire
                        604800 )       ; Negative Cache TTL
;
@       IN      NS      pasopati.it42.com.
@       IN      A       10.84.1.4     ; IP Kotalingga
www     IN      CNAME   pasopati.it42.com.
ns1		IN		A		10.84.1.2		; delegasi IP Majapahit
panah	IN		NS		ns1				; subdomain IP delegasi' > /etc/bind/jarkom/pasopati.it42.com

service bind9 restart
```

## Script Konfigurasi Domain Panah

### Script Konfigurasi
```bash
echo 'zone "panah.pasopati.it42.com" {
    type master;
    notify yes;
    file "/etc/bind/panah/panah.pasopati.it42.com";
};' >> /etc/bind/named.conf.local

mkdir /etc/bind/panah
cp /etc/bind/db.local /etc/bind/panah/panah.pasopati.it42.com

echo '
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     panah.pasopati.it42.com. root.panah.pasopati.it42.com. (
                        2024100201      ; Serial
                        604800         ; Refresh
                        86400         ; Retry
                        2419200         ; Expire
                        604800 )       ; Negative Cache TTL
;
@       IN      NS      panah.pasopati.it42.com.
@       IN      A       10.84.1.4     ; IP Kotalingga
www     IN      CNAME   panah.pasopati.it42.com.' > /etc/bind/panah/panah.pasopati.it42.com

service bind9 restart
```

## Modifikasi Konfigurasi BIND di Master dan Slave

### Konfigurasi `named.conf.options`
```bash
echo '
options {
	directory "/var/cache/bind";
	allow-query { any; };
	auth-nxdomain no; #conform to RFC1035
	listen-on-v6 { any; };
};' > /etc/bind/named.conf.options
```

### Modifikasi `named.conf.local` untuk Slave
```bash
zone "sudarsana.it42.com" {
    type slave;
    masters { 10.84.1.2; };
    file "/var/lib/bind/sudarsana.it42.com";
};

zone "pasopati.it42.com" {
    type slave;
    masters { 10.84.1.2; };
    file "/var/lib/bind/pasopati.it42.com";
};

zone "rujapala.it42.com" {
    type slave;
    masters { 10.84.1.2; };
    file "/var/lib/bind/rujapala.it42.com";
};

zone "panah.pasopati.it42.com" {
    type master;
    file "/etc/bind/panah/panah.pasopati.it42.com";
};
```

### Restart Layanan BIND
```bash
service bind9 restart
```

### Uji Koneksi
Ping `panah.pasopati.it42.com` untuk memastikan konfigurasi berhasil.

## No 10
Markas juga meminta catatan kapan saja meme brain rot akan dijatuhkan, maka buatlah subdomain baru di subdomain panah yaitu log.panah.pasopati.xxxx.com serta aliasnya www.log.panah.pasopati.xxxx.com yang juga mengarah ke Kotalingga.
Ubah file `/etc/bind/panah/pasopati.it42.com` menjadi seperti berikut:
```bash
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     panah.pasopati.it42.com. root.panah.pasopati.it42.com. (
                              2         ; Serial
                         604800         ; Refresh
                          864

00         ; Retry
                       2419200         ; Expire
                          604800 )       ; Negative Cache TTL
;
@       IN      NS      panah.pasopati.it42.com.
@       IN      A       10.84.1.4     ; IP Kotalingga
www     IN      CNAME   panah.pasopati.it42.com.
```




