# Praktikum-Modul-2-Jarkom-E01
Anggota
1. 5025201002 - Tegar Ganang Satrio Priambodo
2. 5025201042 - Cindi Dwi Pramudita
3. 05111940007001 - Afdhal Ma'ruf Lukman


## Narasi Pendahuluan 
Twilight (〈黄昏 (たそがれ) 〉, <Tasogare>) adalah seorang mata-mata yang berasal dari negara Westalis. Demi menjaga perdamaian antara Westalis dengan Ostania, Twilight dengan nama samaran Loid Forger (ロイド・フォージャー, Roido Fōjā) di bawah organisasi WISE menjalankan operasinya di negara Ostania dengan cara melakukan spionase, sabotase, penyadapan dan kemungkinan pembunuhan. Berikut adalah peta dari negara Ostania:

![image](https://user-images.githubusercontent.com/85062827/198755949-923c92d2-dcb5-4571-bfb6-7d74f71e6d0a.png)

## Soal 1
#### WISE akan dijadikan sebagai DNS Master, Berlint akan dijadikan DNS Slave, dan Eden akan digunakan sebagai Web Server. Terdapat 2 Client yaitu SSS, dan Garden. Semua node terhubung pada router Ostania, sehingga dapat mengakses internet
#### Jawaban
Langkah pertama yaitu membuat topologi. Kemudian menyetting network tiap node pada fitur `edit network configuration` yang bisa diakses di menu `Configure`. Settingan awal dihapus dan diganti dengan konfigurasi sebagai berikut
- Ostania
```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
    address 10.22.1.1
    netmask 255.255.255.0

auto eth2
iface eth2 inet static
    address 10.22.2.1
    netmask 255.255.255.0

auto eth3
iface eth3 inet static
    address 10.22.3.1
    netmask 255.255.255.0
```
- Wise
```
auto eth0
iface eth0 inet static
    address 10.22.3.2
    netmask 255.255.255.0
  gateway 10.22.3.1
```
- SSS
```
auto eth0
iface eth0 inet static
    address 10.22.1.2
    netmask 255.255.255.0
  gateway 10.22.1.1
```
- Garden
```
auto eth0
iface eth0 inet static
    address 10.22.1.3
    netmask 255.255.255.0
  gateway 10.22.1.1
```
- Berlint
```
auto eth0
iface eth0 inet static
    address 10.22.2.2
    netmask 255.255.255.0
  gateway 10.22.2.1
```
- Eden
```
auto eth0
iface eth0 inet static
    address 10.22.2.3
    netmask 255.255.255.0
  gateway 10.22.2.1
```
#### Ostania
Setelah dikonfigurasikan kemudian lakukan restart semua node. lalu jalankan command berikut pada router `Ostania` untuk pengaturan lalu lintas komputer
```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.22.0.0/16
```
(Catatan: Prefix IP yang digunakan sesuai Prefix IP Kelompok. untuk Kelompok E1 menggunakan Prefix IP 10.22)

Setelah itu ketikkan command ini pada router `Ostania` untuk melihat IP DNS
```
cat /etc/resolv.conf
```
Nameserver akan muncul dan ini dapat digunakan pada konfigurasi selanjutnya

![image](https://user-images.githubusercontent.com/85062827/198767649-5250f8dd-76e0-4a29-a75d-cb48f8545ac9.png)
#### Konfigurasi semua node (kecuali Ostania)
Agar semua node yang berhubungan dapat mengakses internet, jalankan perintah beriku dan gunakan IP DNS dari `Ostania`
```
echo nameserver 192.168.122.1 > /etc/resolv.conf
```
Restart semua node dan lakukan testing apakah semua node sudah terkoneksi dengan internet. testing berupa ping ke google.com
![image](https://user-images.githubusercontent.com/85062827/198770406-5d26d221-7a28-4e97-bdd1-f3c4ccee2f4f.png)

## Soal 2
#### Untuk mempermudah mendapatkan informasi mengenai misi dari Handler, bantulah Loid membuat website utama dengan akses wise.yyy.com dengan alias www.wise.yyy.com pada folder wise 
#### Jawaban
Lakukan instalasi bind9 terlebih dahulu pada `Wise` dengan update package dan jalankan command berikut
```
apt-get update
apt-get install bind9 -y
```
Setelah instalasi, buat domain wise.e01.com pada `Wise`
```
echo  'zone "wise.e01.com" {
        type master;
        file "/etc/bind/wise/wise.e01.com";
};'  > /etc/bind/named.conf.local
```
Kemudian buat folder baru pada /etc/bind
```
mkdir /etc/bind/wise
```
Lakukan copy file db.local kedalam folder wise dan ubah namanya menjadi wise.e01.com
```
cp /etc/bind/db.local /etc/bind/wise/wise.e01.com
```
Buka file `wise.e01.com` lalu ubah configurasinya
```
echo  '
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     wise.e01.com. root.wise.e01.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@                       IN      NS      wise.e01.com.
@                       IN      A       10.22.3.2
www                     IN      CNAME   wise.e01.com.
eden                    IN      A       10.22.2.3
www.eden                IN      CNAME   eden
' > /etc/bind/wise/wise.e01.com
```
Didalam kongurasi sudah ditambahkan record CNAME www.wise.e01.com untuk membuat ailas yang akan mengarahkan domain ke wise.e01.com

Setelah itu restart bind9
```
service bind9 restart
```

#### SSS atau Garden
Testing dengan menambahkan `nameserver 10.22.3.2` IP Wise pada `SSS` dan `Garden` untuk memerikasa apakah wise.e01.com atau alias www.wise.e01.com sudah dapat diakses.
```
echo 'nameserver 10.22.3.2' > /etc/resolv.conf
ping wise.e01.com -c 5
ping www.wise.e01.com -c 5
```
jika sukses maka akan memunculkan sebagai berikut

![image](https://user-images.githubusercontent.com/85062827/198781176-96a1106d-2b24-4364-b3b4-a95cb0059a3b.png)


## Soal 3
#### Setelah itu ia juga ingin membuat subdomain eden.wise.yyy.com dengan alias www.eden.wise.yyy.com yang diatur DNS-nya di WISE dan mengarah ke Eden
#### Jawaban
#### Wise
Buka file wise.e01.com dan edit seperti berikut
```
echo  '
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     wise.e01.com. root.wise.e01.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@                       IN      NS      wise.e01.com.
@                       IN      A       10.22.3.2
www                     IN      CNAME   wise.e01.com.
eden                    IN      A       10.22.2.3
www.eden                IN      CNAME   eden
' > /etc/bind/wise/wise.e01.com
```
#### SSS atau Garden
lakukan testing pada `sss` dan `Garden` apakah eden.wise.e01.com atau www.eden.wise.e01.com dapat diakses. jika sukses, maka akan memunculkan hasil seperti berikut.

![image](https://user-images.githubusercontent.com/85062827/198790731-26c29310-611a-4375-945c-8fda840c78a0.png)

## Soal 4
#### Buat juga reverse domain untuk domain utama
#### Jawaban
#### Wise
Edit file /etc/bind/named.conf.local pada `Wise` dan tambahkan konfigurasi berikut. Tambahkan reverse dari 3 bytes awal dari IP yang ingin dilakukan Reverse DNS. Dalam hal ini IP 10.22.3 untuk IP dari record sehingga reverse-nya adalah 3.22.10.
```
echo  'zone "3.22.10.in-addr.arpa" {
    type master;
    file "/etc/bind/wise/3.22.10.in-addr.arpa";
};'  > /etc/bind/named.conf.local

```
Copy file db.local ke dalam folder wise dan ubah namanya menjadi 3.22.10.in-addr.arpa.
```
cp /etc/bind/db.local /etc/bind/wise/3.22.10.in-addr.arpa
```
buka file 3.22.10.in-addr.arpa dan edit seperti konfigurasi berikut
```
echo  '
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     wise.e01.com. root.wise.e01.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
3.22.10.in-addr.arpa. IN      NS      wise.e01.com.
2       IN      PTR     wise.e01.com.
' > /etc/bind/wise/3.22.10.in-addr.arpa
service bind9 restart
```
#### SSS atau Garden
Untuk memeriksa apakah konfigurasi sudah benar atau beluum, bisa lakukan perintah berikut
```
echo  '
nameserver 192.168.122.1
'  > /etc/resolv.conf
apt-get update
apt-get install dnsutils -y
echo  '
nameserver 10.22.3.2
'  > /etc/resolv.conf
host -t PTR 10.22.3.2
```

![image](https://user-images.githubusercontent.com/85062827/198800770-cf0cd06f-da26-4b62-b159-d2fe11fe266e.png)

## Soal 5
#### Agar dapat tetap dihubungi jika server WISE bermasalah, buatlah juga Berlint sebagai DNS Slave untuk domain utama
#### Wise
Modifikasi konfigurasi berikut pada /etc/bind/named.conf.local di `Wise`.
```
echo  'zone "wise.e01.com" {
        type master;
	   notify yes;
	   also-notify { 10.22.2.2; };
	   allow-transfer { 10.22.2.2; };
        file "/etc/bind/wise/wise.e01.com";
};' > /etc/bind/named.conf.local
```
Restart bind9.
```
service bind9 restart
```
#### Berlint
Melakukan instalasi bind9 pada Berlint dengan update package list. Command yang dijalankan adalah sebagai berikut.
```
apt-get update
apt-get install bind9 -y
```
Tambahkan konfigurasi berikut pada /etc/bind/named.conf.local di Berlint.
```
echo  'zone "wise.e01.com" {
        type slave;
	   masters { 10.22.3.2; };
   file "/var/lib/bind/wise.e01.com";
};' > /etc/bind/named.conf.local
```
Restart bind9.
```
service bind9 restart
```
#### Wise
Lakukan testing pada SSS dan Garden untuk mengecek apakah DNS Slave berhasil dibuat pada Berlint. untuk itu Stop service bind9 pada Wise
```
service bind9 stop
```
#### SSS atau Garden
Pada `SSS` dan `Garden' ditambahkan nameserver `Berlint` yaitu 10.22.2.2 pada /etc/resolv.conf.

```
echo  '
nameserver 10.22.3.2
nameserver 10.22.2.2
'  > /etc/resolv.conf
ping wise.e01.com -c 5
```
	
hasilnya seperti berikut

![image](https://user-images.githubusercontent.com/85062827/198814003-38622ff9-0870-4691-bda4-dda0bca6f49a.png)

#### Jawaban
## Soal 6
#### Karena banyak informasi dari Handler, buatlah subdomain yang khusus untuk operation yaitu operation.wise.yyy.com dengan alias www.operation.wise.yyy.com yang didelegasikan dari WISE ke Berlint dengan IP menuju ke Eden dalam folder operation
#### Jawaban
## Soal 7
#### Untuk informasi yang lebih spesifik mengenai Operation Strix, buatlah subdomain melalui Berlint dengan akses strix.operation.wise.yyy.com dengan alias www.strix.operation.wise.yyy.com yang mengarah ke Eden 
#### Jawaban
## Soal 8
#### Setelah melakukan konfigurasi server, maka dilakukan konfigurasi Webserver. Pertama dengan webserver www.wise.yyy.com. Pertama, Loid membutuhkan webserver dengan DocumentRoot pada /var/www/wise.yyy.com
#### Jawaban
## Soal 9
#### Setelah itu, Loid juga membutuhkan agar url www.wise.yyy.com/index.php/home dapat menjadi menjadi www.wise.yyy.com/home
#### Jawaban
## Soal 10
#### Setelah itu, pada subdomain www.eden.wise.yyy.com, Loid membutuhkan penyimpanan aset yang memiliki DocumentRoot pada /var/www/eden.wise.yyy.com
#### Jawaban
