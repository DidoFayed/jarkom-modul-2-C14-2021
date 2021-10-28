# Jarkom-Modul-2-C14-2021
Praktikum Jaringan Komputer Modul 2 - DNS dan Web Server

### Nama Anggota Kelompok:
1. 05111940000059     Dido Fabian Fayed <br>
2. 05111940000074	    Nur Ahmad Khatim <br>
3. 05111940000162	    Ramadhan Arif Hardijansyah

## Soal 1
Ingin dibuat sebuah peta topology GNS3, dimana:
- EniesLobby akan dijadikan sebagai DNS Master.
- Water7 akan dijadikan DNS Slave.
- Skypie akan digunakan sebagai Web Server.
- Terdapat 2 Client yaitu Loguetown, dan Alabasta.
- Semua node terhubung pada router Foosha, sehingga dapat mengakses internet.

### Cara Pengerjaan
Dilakukan modifikasi terhadap topologi yang telah dibuat di modul GNS3 dengan menambahkan node baru, yaitu Skypie.
Topologi ini terdiri dari node ethernet switch dan ubuntu, dan membuat hubungan antar node serta nama-nama dari node hingga tampak seperti di gambar.
![ssmodul2](https://github.com/DidoFayed/jarkom-modul-2-C14-2021/blob/main/ssmodul2/1_1_Topology.png)

Lalu, dilakukan pengisian settingan network node Skypie dengan fitur `Edit network configuration` sebagai berikut.

- Skypie
```
auto eth0
iface eth0 inet static
	address 10.21.2.4
	netmask 255.255.255.0
	gateway 10.21.2.1
```

## Soal 2
Membuat website utama dengan mengakses franky.yyy.com dengan alias www.franky.yyy.com pada folder kaizoku.
### Cara Pengerjaan
- Buka node Skypie dan update package list.
```
apt-get update
```

- Install aplikasi bind9 pada node Skypie dengan command sesbagai berikut.
```
apt-get install bind9 -y
```

#### Pembuatan domain franky.c14.com
- Lakukan command pada node Skypie sebagai berikut.
```
nano /etc/bind/named.conf.local
```

- Isikan configurasi domain **franky.c14.com** dengan syntax sebagai berikut.
```
zone "franky.c14.com" {
	type master;
	file "/etc/bind/kaizoku/franky.c14.com";
};
```
- Buat folder **kaizoku** dalam `/etc/bind`.
```
mkdir /etc/bind/kaizoku
```

-  Copy file **db.local** pada path `/etc/bind` ke dalam folder **kaizoku** yang baru saja dibuat dan ubah namanya menjadi **franky.c14.com**.
```
cp /etc/bind/db.local /etc/bind/kaizoku/franky.c14.com
```

- Buka file **franky.c14.com** dan edit isinya sehingga tampak seperti gambar berikut, dengan IP Skypie masing-masing kelompok.
```
nano /etc/bind/kaizoku/franky.c14.com
```
Isi dengan:
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     franky.c14.com. root.franky.c14.com. (
                        2020102801      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      franky.c14.com.
@       IN      A       10.40.2.2       ; IP EniesLobby
```

- Restart bind9 dengan perintah
```
service bind9 restart
```

#### Setting nameserver pada client
Dilakukan pengaturan nameserver pada client agar domain yang telah dibuat dapat dikenali oleh client.
- Pada client Loguetown dan Alabasta arahkan nameserver menuju IP Skypie dengan mengedit file `/etc/resolv.conf` dengan command.
```
nano /etc/resolv.conf
```
Edit isinya sehingga terlihat seperti berikut.
```
# nameserver 192.168.122.1

nameserver 10.21.2.4 # ip Skypie
```

- Untuk mencoba koneksi DNS, lakukan ping domain **franky.c14.com** pada client Loguetown dan Alabasta.
```
ping franky.c14.com
```
![ssmodul2](https://github.com/DidoFayed/jarkom-modul-2-C14-2021/blob/main/ssmodul2/2_1_PingFrankyLoguetownAlabasta.png)

#### Record CNAME
Record CNAME adalah sebuah record yang membuat alias name dan mengarahkan domain ke alamat/domain lain.
- Buka file **franky.c14.com** pada server Skypie dan tambahkan konfigurasi seperti berikut.
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     franky.c14.com. root.franky.c14.com. (
                        2020102801      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      franky.c14.com.
@       IN      A       10.21.2.4       ; IP Skypie
WWW     IN      CNAME   franky.c14.com.
```

- Kemudian restart bind9 dengan command
```
service bind9 restart
```

- Lalu cek dengan `host -t CNAME www.franky.c14.com` atau `ping www.franky.c14.com`. Hasilnya harus mengarah ke host dengan IP Skypie.
![ssmodul2](https://github.com/DidoFayed/jarkom-modul-2-C14-2021/blob/main/ssmodul2/2_2_HostPingWWW.png)

## Soal 3
Buat subdomain super.franky.c14.com dengan alias www.super.franky.c14.com yang diatur DNS nya di EniesLobby dan mengarah ke Skypie
### Cara Pengerjaan

## Soal 4
Membuat reverse domain untuk domain utama.
### Cara Pengerjaan
#### Reverse DNS (Record PTR)
Digunakan untuk menerjemahkan IP ke alamat domain yang sudah diterjemahkan sebelumnya.
- Edit file `/etc/bind/named.conf.local` pada Skypie.
```
nano /etc/bind/named.conf.local
```
- Tambahkan konfigurasi berikut ke dalam file `/etc/bind/named.conf.local`. Tambahkan reverse tiga byte awal dari IP yang ingin dilakukan reverse. Karena digunakan IP `10.21.2` untuk IP dari records, maka reverse-nya adalah `2.21.10`
```
zone "2.21.10.in-addr.arpa" {
    type master;
    file "/etc/bind/kaizoku/2.21.10.in-addr.arpa";
};
```
![ssmodul2](https://github.com/DidoFayed/jarkom-modul-2-C14-2021/blob/main/ssmodul2/3_1_AddReverse.png)

- Copy file **db.local** ppada path `/etc/bind` ke dalam folder kaizoku yang baru saja dibuat dan ubah namanya menjadi **2.21.10.in-addr.arpa**
```
cp /etc/bind/db.local /etc/bind/kaizoku/2.21.10.in-addr.arpa
```
- Edit file, masukkan command `nano /etc/bind/kaizoku/2.21.10.in-addr.arpa` dan ubah sehingga menjadi sebagai berikut.
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     franky.c14.com. root.franky.c14.com. (
                        2021102801      ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
2.21.10.in-addr.arpa    IN      NS      franky.c14.com.
4                       IN      PTR     franky.c14.com. ; BYTE KE-4 IP Skypie
```

- Kemudian restart bind9 dengan command
```
service bind9 restart
```

- Untuk mengecek konfigurasi, lakukan command berikut pada client Loguetown.
```
// Ubah dulu nameserver di /etc/resolv.conf menjadi sama dengan nameserver dari Foosha agar dapat tersambung ke internet
apt-get update
apt-get install dnsutils

// Ubah kembali nameserver agar tersambung dengan Skypie
host -t PTR 10.21.2.4
```
Screenshot:
![ssmodul2](https://github.com/DidoFayed/jarkom-modul-2-C14-2021/blob/main/ssmodul2/3_2_HostPTR.png)


## Kendala yang Dialami Selama Pengerjaan
Kendala yang Dialami Selama Pengerjaan
1. Beberapa teman yang menggunakan sistem operasi selain Linux mengalami kesulitan dalam mengerjakan, terdapat error ketika menjalankan GNS3, dan lain-lain.
2. Ketika mengerjakan pada saat praktikum, sempat melakukan konfigurasi pada folder dan Web Server yang tidak sesuai, salah menempatkan pada folder **jarkom**, yang seharusnya **kaizoku**. Sehingga butuh untuk membuat ulang folder dan mengkonfigurasi script-script dari awal.

