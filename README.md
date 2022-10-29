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
