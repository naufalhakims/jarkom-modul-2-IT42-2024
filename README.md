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



