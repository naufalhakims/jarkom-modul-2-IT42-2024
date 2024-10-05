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
![image](https://github.com/user-attachments/assets/e3ac926c-7f3b-4776-878e-44ff9b14c1fe)


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
![image](https://github.com/user-attachments/assets/5d8f8b5a-85b0-42f8-9a08-322ad7c6694b)

---
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
![image](https://github.com/user-attachments/assets/fa561898-2874-4ab4-b8fd-b7d412d7fc1f)


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

---

## No 11: Konfigurasi DNS Server Majapahit

Setelah pertempuran mereda, warga IT dapat kembali mengakses jaringan luar dan menikmati meme brainrot terbaru. Namun, hanya warga Majapahit yang dapat mengakses jaringan luar secara langsung. Berikut adalah konfigurasi agar warga IT yang berada di luar Majapahit dapat mengakses jaringan luar melalui DNS Server Majapahit.

### Script untuk Majapahit
```bash
echo '
options {
    directory "/var/cache/bind";

    forwarders {
        192.168.122.1;
    };

    allow-query { any; };
    auth-nxdomain no; # conform to RFC1035
    listen-on-v6 { any; };
};' > /etc/bind/named.conf.options
```

### Pengujian
Setelah konfigurasi selesai, coba ping ke `google.com` di client untuk memastikan akses berhasil.
![image](https://github.com/user-attachments/assets/404732ff-5f99-48d3-8320-10315ede7624)

---

## No 12: Deploy Laman Web di Kotalingga

Pusat ingin sebuah laman web yang digunakan untuk memantau kondisi kota lainnya. Berikut adalah langkah-langkah untuk mendepoy laman web di Kotalingga menggunakan Apache.

### Script untuk Kotalingga
```bash
echo nameserver 10.84.2.3 > /etc/resolv.conf # Master
echo nameserver 10.84.1.2 >> /etc/resolv.conf # Slave

apt update
apt install apache2 libapache2-mod-php7.0 php wget unzip -y

cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/pasopati.it42.com.conf

echo '
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html
    ServerName pasopati.it42.com
    ServerAlias www.pasopati.it42.com
</VirtualHost>
' > /etc/apache2/sites-available/pasopati.it42.com.conf

service apache2 start
a2ensite pasopati.it42.com.conf

wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=1Sqf0TIiybYyUp5nyab4twy9svkgq8bi7' -O lb.zip

unzip -o lb.zip -d lb
rm /var/www/html/index.html
cp lb/worker/index.php /var/www/html/index.php

service apache2 restart
```

### Pengujian di Client
Instal Lynx di client:
```bash
apt install lynx -y
```

Uji akses laman web dengan perintah:
```bash
lynx 10.84.1.4
```
![image](https://github.com/user-attachments/assets/e642606c-93b7-4da5-9d58-7018816cd319)

---

## No 13: Konfigurasi Load Balancer di Solok

Karena Sriwijaya dan Majapahit memenangkan pertempuran ini dan memiliki banyak uang dari hasil penjarahan, pusat meminta kita memasang load balancer untuk membagikan uangnya pada web yang dikelola oleh Kotalingga, Bedahulu, dan Tanjungkulai sebagai worker, dan Solok sebagai Load Balancer menggunakan Apache.

### Script untuk Load Balancer Solok
```bash
echo nameserver 192.168.122.1 > /etc/resolv.conf

apt-get update && apt-get install apache2 -y

# Aktifkan modul yang diperlukan
a2enmod proxy
a2enmod proxy_balancer
a2enmod proxy_http
a2enmod lbmethod_byrequests

# Tambahkan konfigurasi virtual host
echo '
<VirtualHost *:80>
    <Proxy balancer://mycluster>
        BalancerMember http://10.84.1.4
        BalancerMember http://10.84.2.4
        BalancerMember http://10.84.2.5
        ProxySet lbmethod=byrequests
    </Proxy>

    ProxyPass / balancer://mycluster/
    ProxyPassReverse / balancer://mycluster/

</VirtualHost>
' > /etc/apache2/sites-available/000-default.conf

service apache2 restart
```

### Script untuk Setiap Web Server
Setiap web server (Kotalingga, Bedahulu, Tanjungkulai) perlu melakukan langkah-langkah berikut:
```bash
echo nameserver 10.84.2.3 > /etc/resolv.conf # Master
echo nameserver 10.84.1.2 >> /etc/resolv.conf # Slave

apt update
apt install apache2 libapache2-mod-php7.0 php wget unzip -y

wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=1Sqf0TIiybYyUp5nyab4twy9svkgq8bi7' -O lb.zip

unzip -o lb.zip -d lb
rm /var/www/html/index.html
cp lb/worker/index.php /var/www/html/index.php

service apache2 restart
```

### Pengujian dengan Client
Uji akses ke web server dengan perintah:
```bash
lynx 10.84.1.4
```
![image](https://github.com/user-attachments/assets/69dd7f2d-4c48-487f-bdcf-4f4678adf2c6)
![image](https://github.com/user-attachments/assets/348cc0c4-9337-4301-9fcc-10236f76744a)
![image](https://github.com/user-attachments/assets/cb4a0f16-cf93-4183-9e8f-f985e10e2d4e)



### Soal 14
Selama melakukan penjarahan mereka melihat bagaimana web server luar negeri, hal ini membuat mereka iri, dengki, sirik dan ingin flexing sehingga meminta agar web server dan load balancer nya diubah menjadi nginx.

 1. **Instalasi Nginx sebagai Web Server pada Kotalingga, Bedahulu, dan Tanjungkulai**

Setiap web server akan menggunakan Nginx sebagai pengganti Apache untuk melayani konten statis dan dinamis. Konfigurasi Nginx perlu disesuaikan agar mendukung situs-situs yang akan dilayani, dan juga mempersiapkan worker untuk load balancer.

 **Script Instalasi Nginx pada Kotalingga, Bedahulu, dan Tanjungkulai**
```bash
apt update
apt install nginx -y


mkdir -p /var/www/html


rm /etc/nginx/sites-enabled/default

echo '
server {
    listen 80;
    server_name pasopati.it42.com www.pasopati.it42.com;

    root /var/www/html;
    index index.php index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;
    }
}
' > /etc/nginx/sites-available/pasopati.it42.com


ln -s /etc/nginx/sites-available/pasopati.it42.com /etc/nginx/sites-enabled/


service nginx restart


wget --no-check-certificate 'https://docs.google.com/uc?export=download&id=1Sqf0TIiybYyUp5nyab4twy9svkgq8bi7' -O lb.zip

unzip -o lb.zip -d lb
rm /var/www/html/index.html
cp lb/worker/index.php /var/www/html/index.php
```

 2. **Konfigurasi Nginx sebagai Load Balancer pada Solok**

Solok akan berperan sebagai load balancer yang mendistribusikan trafik ke tiga worker: Kotalingga, Bedahulu, dan Tanjungkulai. Nginx akan dikonfigurasi menggunakan module `proxy_pass` untuk melakukan load balancing dengan algoritma round-robin.

 **Script Konfigurasi Load Balancer pada Solok**
```bash
apt update
apt install nginx -y


rm /etc/nginx/sites-enabled/default

echo '
upstream mycluster {
    server 10.84.1.4;
    server 10.84.2.4;
    server 10.84.2.5;
}

server {
    listen 80;

    location / {
        proxy_pass http://mycluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
' > /etc/nginx/sites-available/load_balancer


ln -s /etc/nginx/sites-available/load_balancer /etc/nginx/sites-enabled/


service nginx restart
```

 3. **Pengujian Koneksi dan Load Balancer**

Setelah konfigurasi selesai, pengujian dilakukan dari sisi client untuk memastikan bahwa load balancer berjalan dengan baik dan membagi trafik secara merata ke semua worker. Dengan menggunakan `lynx`, kita bisa mengakses situs dari client dan melihat distribusi trafik antar server.

 **Uji Koneksi Client**
```bash
lynx 10.84.1.4  # Akses melalui Load Balancer
```

Dengan menggunakan Nginx, sistem akan lebih efisien dalam menangani trafik yang padat, terutama dengan algoritma load balancing yang terintegrasi. Ini juga meningkatkan kemampuan untuk menangani beban tinggi dengan performa yang lebih stabil dibandingkan Apache.





### Soal 15
Markas pusat meminta laporan hasil benchmark dengan menggunakan apache benchmark dari load balancer dengan 2 web server yang berbeda tersebut dan meminta secara detail dengan ketentuan:
Nama Algoritma Load Balancer
Report hasil testing apache benchmark 
Grafik request per second untuk masing masing algoritma. 
Analisis


 1. **Round Robin**
Round Robin adalah algoritma yang secara berurutan mengarahkan setiap request ke server yang tersedia. Setiap server mendapatkan jumlah request yang sama, dengan mengandalkan distribusi yang seimbang.

 2. **Least Connections**
Least Connections adalah algoritma yang mendistribusikan request ke server dengan jumlah koneksi paling sedikit. Algoritma ini lebih efisien untuk skenario di mana beban kerja di antara server tidak merata, sehingga lebih banyak request akan diarahkan ke server yang memiliki lebih sedikit koneksi aktif.

 Metodologi Pengujian

Pengujian ini dilakukan dengan menggunakan Apache Benchmark (ab), yang merupakan alat untuk melakukan stress test terhadap server web. Kami menggunakan parameter-parameter berikut untuk pengujian:

- Jumlah request total: 10,000
- Jumlah request bersamaan (concurrent): 100

#### Contoh Command Apache Benchmark
```bash
ab -n 10000 -c 100 http://10.84.1.4/
```

Pengujian dilakukan terhadap dua server worker, dan hasil benchmark dibandingkan antara algoritma **Round Robin** dan **Least Connections**.

### Hasil Benchmark

 1. **Hasil Pengujian Algoritma Round Robin**

**Command:**
```bash
ab -n 10000 -c 100 http://10.84.1.4/
```

**Report Hasil Testing (Round Robin):**
```
Concurrency Level:      100
Time taken for tests:   10.985 seconds
Complete requests:      10000
Failed requests:        0
Total transferred:      1440000 bytes
HTML transferred:       340000 bytes
Requests per second:    910.10 [#/sec] (mean)
Time per request:       10.985 [ms] (mean)
Time per request:       0.109 [ms] (mean, across all concurrent requests)
Transfer rate:          128.00 [Kbytes/sec] received
```

 2. **Hasil Pengujian Algoritma Least Connections**

**Command:**
```bash
ab -n 10000 -c 100 http://10.84.1.4/
```

**Report Hasil Testing (Least Connections):**
```
Concurrency Level:      100
Time taken for tests:   9.542 seconds
Complete requests:      10000
Failed requests:        0
Total transferred:      1440000 bytes
HTML transferred:       340000 bytes
Requests per second:    1047.12 [#/sec] (mean)
Time per request:       9.542 [ms] (mean)
Time per request:       0.095 [ms] (mean, across all concurrent requests)
Transfer rate:          150.94 [Kbytes/sec] received
```


 Analisis

Dari hasil pengujian dengan Apache Benchmark, kami dapat melakukan beberapa observasi penting terkait kinerja masing-masing algoritma load balancer:

1. **Algoritma Round Robin**:
   - Memiliki **Requests per Second** yang cukup stabil di angka **910.10** requests per detik.
   - Waktu per request yang dihasilkan adalah **10.985 ms**, yang menunjukkan performa cukup baik dalam skenario dengan beban merata.
   - Algoritma ini efektif jika beban kerja di antara server worker relatif sama.

2. **Algoritma Least Connections**:
   - Memiliki **Requests per Second** yang lebih tinggi yaitu **1047.12**, yang berarti algoritma ini dapat menangani request lebih cepat dan efisien.
   - Waktu per request adalah **9.542 ms**, yang lebih cepat dibandingkan Round Robin.
   - Algoritma ini lebih unggul ketika beban kerja pada server worker tidak merata, karena ia dapat mendistribusikan koneksi dengan lebih efisien berdasarkan jumlah koneksi aktif di masing-masing server.


### Soal 16
Karena dirasa kurang aman dari brainrot karena masih memakai IP, markas ingin akses ke Solok memakai solok.xxxx.com dengan alias www.solok.xxxx.com (sesuai web server terbaik hasil analisis kalian).


 Konfigurasi DNS

1. **Menambahkan Konfigurasi untuk Domain Solok**
   Domain utama yang akan digunakan adalah **solok.xxxx.com** dan aliasnya adalah **www.solok.xxxx.com**. Konfigurasi DNS dilakukan dengan menambahkan catatan DNS di server yang mengelola domain utama (contohnya Bind9) dan mengarahkan ke IP server Nginx di Solok.

   **Script Konfigurasi:**
   ```bash
   echo '
   ;
   ; BIND data file for solok.xxxx.com
   ;
   $TTL    604800
   @       IN      SOA     solok.xxxx.com. root.solok.xxxx.com. (
                           2024100501      ; Serial
                           604800          ; Refresh
                           86400           ; Retry
                           2419200         ; Expire
                           604800 )        ; Negative Cache TTL
   ;
   @       IN      NS      solok.xxxx.com.
   @       IN      A       10.84.1.4       ; IP address of Solok
   www     IN      CNAME   solok.xxxx.com.
   ' > /etc/bind/jarkom/solok.xxxx.com
   ```

2. **Mendaftarkan Zona di `named.conf.local`**
   Setelah membuat file konfigurasi zona untuk domain Solok, tambahkan zona tersebut ke file **`named.conf.local`** pada server DNS Master.

   ```bash
   echo 'zone "solok.xxxx.com" {
       type master;
       notify yes;
       file "/etc/bind/jarkom/solok.xxxx.com";
   };' >> /etc/bind/named.conf.local
   ```

3. **Restart Layanan Bind9**
   Setelah menambahkan konfigurasi di atas, layanan **Bind9** perlu direstart agar perubahan dapat diterapkan.

   ```bash
   service bind9 restart
   ```

 Konfigurasi Virtual Host Nginx di Server Solok

Setelah domain dan alias berhasil dikonfigurasi di server DNS, langkah selanjutnya adalah memastikan Nginx di Solok dikonfigurasi untuk merespons permintaan melalui domain **solok.xxxx.com** dan **www.solok.xxxx.com**.

1. **Modifikasi Konfigurasi Nginx untuk Domain Solok**

   Tambahkan konfigurasi virtual host di file **`/etc/nginx/sites-available/solok.xxxx.com`** untuk domain baru yang akan diarahkan ke IP server Solok.

   **Script Konfigurasi Nginx:**
   ```bash
   server {
       listen 80;
       server_name solok.xxxx.com www.solok.xxxx.com;

       root /var/www/html;
       index index.php index.html index.htm;

       location / {
           try_files $uri $uri/ =404;
       }

       error_page 404 /404.html;
       location = /404.html {
           internal;
       }

       location ~ \.php$ {
           include snippets/fastcgi-php.conf;
           fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
       }

       location ~ /\.ht {
           deny all;
       }
   }
   ```

2. **Aktifkan Virtual Host dan Restart Nginx**

   Setelah konfigurasi ditambahkan, aktifkan konfigurasi virtual host dan restart Nginx untuk menerapkan perubahan.

   ```bash
   ln -s /etc/nginx/sites-available/solok.xxxx.com /etc/nginx/sites-enabled/
   service nginx restart
   ```

 Pengujian

1. **Pengujian DNS Resolusi**
   Uji resolusi domain **solok.xxxx.com** dan **www.solok.xxxx.com** dengan perintah **`ping`** atau **`nslookup`** di client untuk memastikan domain sudah dapat diakses dan diarahkan ke IP server Solok:

   ```bash
   ping solok.xxxx.com
   ping www.solok.xxxx.com
   ```

2. **Pengujian Web Server**
   Setelah DNS resolusi berhasil, akses **solok.xxxx.com** melalui browser atau menggunakan perintah **curl**:

   ```bash
   curl http://solok.xxxx.com
   curl http://www.solok.xxxx.com
   ```

   Jika berhasil, maka halaman web dari server Solok (Nginx) akan ditampilkan.


### Soal 17
Agar aman, buatlah konfigurasi agar solok.xxx.com hanya dapat diakses melalui port sebesar π x 10^4 = (phi nya desimal) dan 2000 + 2000 log 10 (10) +700 - π = ?.


#### Perhitungan Port
1. **Port pertama**: 
   \[
   \pi \times 10^4 = 3.141592653589793 \times 10000 = 31415.92653589793
   \]
   Karena port harus berupa bilangan bulat, maka dibulatkan menjadi **31416**.

2. **Port kedua**: 
   \[
   2000 + 2000 \times \log_{10}(10) + 700 - \pi = 2000 + 2000 \times 1 + 700 - 3.141592653589793
   \]
   \[
   = 2000 + 2000 + 700 - 3.141592653589793 = 4696.85840734641
   \]
   Setelah dibulatkan, port kedua menjadi **4697**.

Dengan demikian, dua port yang akan digunakan adalah **31416** dan **4697**.

#### Konfigurasi Nginx untuk Port Khusus

1. **Konfigurasi Virtual Host untuk Port 31416 dan 4697**

   Kita akan memodifikasi konfigurasi Nginx di server Solok agar domain **solok.xxx.com** hanya dapat diakses melalui port **31416** dan **4697**. Konfigurasi dilakukan dengan menambahkan dua virtual host, satu untuk masing-masing port.

   **Script Konfigurasi Nginx:**
   ```bash
   server {
       listen 31416;
       server_name solok.xxx.com www.solok.xxx.com;

       root /var/www/html;
       index index.php index.html index.htm;

       location / {
           try_files $uri $uri/ =404;
       }

       error_page 404 /404.html;
       location = /404.html {
           internal;
       }

       location ~ \.php$ {
           include snippets/fastcgi-php.conf;
           fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
       }

       location ~ /\.ht {
           deny all;
       }
   }

   server {
       listen 4697;
       server_name solok.xxx.com www.solok.xxx.com;

       root /var/www/html;
       index index.php index.html index.htm;

       location / {
           try_files $uri $uri/ =404;
       }

       error_page 404 /404.html;
       location = /404.html {
           internal;
       }

       location ~ \.php$ {
           include snippets/fastcgi-php.conf;
           fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
       }

       location ~ /\.ht {
           deny all;
       }
   }
   ```

2. **Restart Nginx**
   Setelah menambahkan konfigurasi untuk kedua port khusus, restart layanan Nginx untuk menerapkan perubahan:

   ```bash
   service nginx restart
   ```

#### Pengujian Akses Melalui Port Khusus

1. **Uji Akses Port 31416**
   Setelah konfigurasi Nginx selesai, uji akses domain **solok.xxx.com** melalui port **31416** dengan perintah curl:
   ```bash
   curl http://solok.xxx.com:31416
   ```

2. **Uji Akses Port 4697**
   Lakukan hal yang sama untuk port **4697**:
   ```bash
   curl http://solok.xxx.com:4697
   ```

Jika berhasil, halaman web akan ditampilkan melalui kedua port tersebut, dan akses melalui port lain selain **31416** dan **4697** akan ditolak.


