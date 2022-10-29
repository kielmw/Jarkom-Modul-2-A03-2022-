# Jarkom-Modul-2-A03-2022
Repository untuk Laporan Resmi Praktikum Modul 2 Jarkom 2022 - A03
* Fayyadh Hafizh
  502520
* Ezekiel Mashal Wicaksono
  5025201140
* Adifa
  502520


### Soal No. 1
Semua node terhubung pada router Ostania, sehingga dapat mengakses internet 

- Masukan Network configuration pada **Ostania**

```auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 192.170.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 192.170.2.1
	netmask 255.255.255.0

auto eth3
iface eth3 inet static
	address 192.170.3.1
	netmask 255.255.255.0
  ```
- Masukan network configuration pada  **Wise , Garden , SSS , Berlint , Eden** . Setiap node kurang lebih memiriki config yang mirip , hanya berbeda pada addres dan gateaway

```
auto eth0
iface eth0 inet static
	address 192.170.2.3
	netmask 255.255.255.0
	gateway 192.170.2.2
```
Contoh ini berlaku pada **Garden**

- Setelah melakukan configuration pada semua node , maka masukan **iptables** di ostania melalui **nano.bashrc**

```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.170.0.0/16
```
- Masukin cofig juga pada nodes di nano.bashrc 

```
echo nameserver 192.168.122.1 > /etc/resolv.conf
```
- Setelah semua sudah maka restart nodes dan semuanya akan bisa ping ke *google.com*
![Nomor 1](https://i.ibb.co/ZmxwTrq/nomor-1.jpg)

### Soal No. 2
membuat website utama dengan akses wise.yyy.com dengan alias www.wise.yyy.com pada folder wise

- Masuk ke web console **WISE**
dan command
```
 apt-get update
apt-get install bind9 -y
```
- Setelah melakukan command tersebut maka masuk ke

```
nano /etc/bind/named.conf.local    
```
dan masukan
```                    
#Isi File
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "wise.a03.com" {
        type master;
        file "/etc/bind/wise/wise.a03.com";
};
```
- kemudian lakukan
```
mkdir /etc/bind/wise
cp /etc/bind/db.local /etc/bind/wise/wise.a03.com
```
- Setelah selesai dilakukan maka kita harus mengedit file tersebut dengan cara
```
nano /etc/bind/wise/wise.a03.com
```
dan diisikan dengan 
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     wise.a03.com. root.wise.a03.com. (
                              2022100601                ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      wise.a03.com.
@       IN      A       192.170.1.2
@       IN      AAAA    ::1
```
- setelah selesai semuanya maka lakukan 
```
service bind9 restart 
```
- Untuk melakukan pengetesan maka ubah namserver dari **SSS** menuju ip dari **Wise** , kemudian coba di ping menuju **wise.a03.com** maka akan didapatkan
![Nomor 2](https://i.ibb.co/zbbqZ66/2.jpg)

### Soal No. 5
- Untuk menjadikan **Berlint** sebagai DNS Slave kita akan mengubah isi dari file **named.conf.local** pada **WISE** menjadi seperti dibawah ini
```
zone "wise.a03.com" {
        type master;
        file "/etc/bind/wise/wise.a03.com";
        allow-transfer { 192.170.3.2; };
};
```
- Lalu kita restart service bind9 dengan command ``` service bind9 restart ```
- Setelah itu, kita akan menginstall bind9 juga pada node **Berlint** dengan command 
```
apt-get update
apt-get install bind9 -y
```
- Pada file **named.conf.local** di **Berlint** kita isi konfigurasi berikut
```
zone "wise.a03.com" {
    type slave;
    masters { 192.170.1.2; }; // Masukan IP EniesLobby tanpa tanda petik
    file "/var/lib/bind/wise.a03.com";
};
```
- Restart service bind9 dengan command ``` service bind9 restart ```
- Setelah semuanya telah dilakukan maka untuk testing kita akan melakukan ping kepada **wise.a03.com** (matikan terlebih dahulu service bind9 pada **WISE** dengan command ``` service bind9 stop ```
- Jika berhasil maka hasilnya akan terlihat seperti ini
![image](https://user-images.githubusercontent.com/72655301/198835253-34831cec-1447-4c48-83fb-c3e3b2406902.png)

### Soal No. 6
- Untuk mendelegasikan domain atau subdomain kita akan menambah konfigurasi pada file **wise.a03.com** di **WISE** seperti berikut
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     wise.a03.com. root.wise.a03.com. (
                     2022100601         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@               IN      NS      wise.a03.com.
@               IN      A       192.170.1.2
www             IN      CNAME   wise.a03.com.
eden            IN      A       192.170. Ke3.3
www.eden        IN      CNAME   eden.wise.a03.com.
ns1             IN      A       192.170.3.2
operation       IN      NS      ns1
@               IN      AAAA    ::1
```
- Kemudian pada file **named.conf.options** di **WISE** kita akan melakukan uncomment pada **dnssec-validation auto;** dan tambahkan ``` allow-query{any;}; ``` dibawahnya.
- Setelah itu, di node **Berlint** pada file **named.conf.options** kita juga akan melakukan uncomment pada **dnssec-validation auto;** dan tambahkan ``` allow-query{any;}; ``` dibawahnya.
- Untuk file **named.conf.local** pada node **Berlint** kita tambahkan konfigurasi seperti berikut
```
zone "operation.wise.a03.com" {
        type master;
        file "/etc/bind/delegasi/operation.wise.a03.com";
};
```
- Buat direktori dengan nama **/etc/bind/delegasi** dan copy file **db.local** ke dalam direktori **/etc/bind/delegasi** dengan nama **operation.wise.a03.com** dengan command berikut
```
mkdir delegasi
cp /etc/bind/db.local /etc/bind/delegasi/operation.wise.a03.com
```
- Isi konfigurasi untuk file **operation.wise.a03.com** seperti dibawah ini
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     operation.wise.a03.com. root.operation.wise.a03.com. (                  
		     2022102601         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      operation.wise.a03.com.
@       IN      A       192.170.3.2
www     IN      CNAME   operation.wise.a03.com.
@       IN      AAAA    ::1
```
- Restart service bind9 dengan menggunakan command ``` service bind9 restart ```
- Lakukan testing dengan melakukan ping pada **operation.wise.a03.com** dan jika berhasil akan terlihat seperti ini
![image](https://user-images.githubusercontent.com/72655301/198835897-52a0592b-b8c6-447a-8bfa-83f67060dbff.png)

### Soal No. 7
- Untuk menambah subdomain, maka kita menggunakan DNS Record tipe A dengan menambahkan nama subdomain pada file **operation.wise.a03.com** pada node **Berlint** sekaligus dengan alias dengan menggunakan DNS Record tipe CNAME seperti dibawah ini
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     operation.wise.a03.com. root.operation.wise.a03.com. (                  
			   2022102601         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      operation.wise.a03.com.
@       IN      A       192.170.3.2
www     IN      CNAME   operation.wise.a03.com.
strix   IN      A       192.170.3.3
www.strix       IN      CNAME   strix.operation.wise.a03.com.
@       IN      AAAA    ::1
```
- Restart service bind9 dengan command ``` service bind9 restart ```
- Setelah itu kita akan melakukan testing dengan melakukan ping pada **strix.operation.wise.a03.com** dan jika berhasil akan terlihat seperti ini
![image](https://user-images.githubusercontent.com/72655301/198836081-bb9604c7-7955-414c-a4a0-c0b3c497481d.png)
