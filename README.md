# Jarkom-modul3-b09-2022

### Anggota Kelompok
| Nama                 | NRP        |
|----------------------|------------|
| Gery Febrian Setyara | 5025201151 |
| Muhammad Yunus       | 5025201171 |
| Hafiz Kurniawan      | 5025201032 |

## Persiapan
Topologi jaringan

![image](https://user-images.githubusercontent.com/92217354/201560120-06528410-cf88-40d0-bc9b-bbd428214ae1.png)


- Konfigurasi `Ostania`
```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 192.177.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 192.177.2.1
	netmask 255.255.255.0


auto eth3
iface eth3 inet static
	address 192.177.3.1
	netmask 255.255.255.0
```
- Konfigurasi `SSS`
```
#auto eth0
#iface eth0 inet static
#        address 192.177.1.2
#        netmask 255.255.255.0
#        gateway 192.177.1.1

 auto eth0
 iface eth0 inet dhcp

```

- Konfigurasi `Garden`
```

#auto eth0
#iface eth0 inet static
#        address 192.177.1.3
#        netmask 255.255.255.0
#        gateway 192.177.1.1

 auto eth0
 iface eth0 inet dhcp


 ```


- Konfigurasi `Wise`
```
auto eth0
iface eth0 inet static
	address 192.177.2.2
	netmask 255.255.255.0
	gateway 192.177.2.1
  
```

 
 - Konfigurasi `Berlint`
 ```
 auto eth0
iface eth0 inet static
	address 192.177.2.3
	netmask 255.255.255.0
	gateway 192.177.2.1
  ```
  
 - Konfigurasi `Westalis`
 ```
 auto eth0
iface eth0 inet static
	address 192.177.2.4
	netmask 255.255.255.0
	gateway 192.177.2.1
  ```
 - Konfigurasi `Eden`
```
#auto eth0
#iface eth0 inet static
#        address 192.177.3.2
#        netmask 255.255.255.0
#        gateway 192.177.3.1

 auto eth0
 iface eth0 inet dhcp
 ```
- Konfigurasi `NewstonCastle`
```
#auto eth0
#iface eth0 inet static
#        address 192.177.3.3
#        netmask 255.255.255.0
#        gateway 192.177.3.1

 auto eth0
 iface eth0 inet dhcp
 ```


- Konfigurasi `KemonoPark`
```
#auto eth0
#iface eth0 inet static
#        address 192.177.3.4
#        netmask 255.255.255.0
#        gateway 192.177.3.1

 auto eth0
 iface eth0 inet dhcp
 ```


## No.1 
Loid bersama Franky berencana membuat peta tersebut dengan kriteria WISE sebagai DNS Server, Westalis sebagai DHCP Server, Berlint sebagai Proxy Server

#### Konfigurasi .bashrc:

- Konfigurasi `Wise(DNS Server)`
```
echo  'nameserver 192.168.122.1' > /etc/resolv.conf
apt-get update
apt-get install bind9 -y
apt-get install nano -y
service bind9 start
```

- Konfigurasi `Westalis(DHCP Server)`
```
echo  'nameserver 192.168.122.1' >> /etc/resolv.conf
apt-get update
apt-get upgrade -y
apt-get install isc-dhcp-server -y
```

- Konfigurasi `Berlint(Proxy Server)`
```
echo  'nameserver 192.168.122.1' >> /etc/resolv.conf
apt-get update
apt-get upgrade -y
apt-get install nano -y
apt-get install squid -y
service squid restart
service squid status
```

## No.2 
dan Ostania sebagai DHCP Relay

- Konfigurasi `Ostania(DHCP Relay)`
```
echo  'nameserver 192.168.122.1' >> /etc/resolv.conf
apt-get update
apt-get upgrade -y
apt-get install nano -y
apt-get install isc-dhcp-relay -y
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 192.177.0.0/16
```
dan melakukan pengisian untuk dhcp server yang ingin di relay dan interfacesnya

![image](https://user-images.githubusercontent.com/92217354/201562633-f7cbcac3-1b19-41b6-87db-c8516ab57034.png)

atau dapat langsung menggunakan 

```
echo '
SERVERS="192.177.2.4"
INTERFACES="eth1 eth2 eth3"
OPTIONS=""
' > /etc/default/isc-dhcp-relay
```
cara manapun sama saja

## No.3 - 6
Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.50 - [prefix IP].1.88 dan [prefix IP].1.120 - [prefix IP].1.155 
```
range 192.177.1.50 192.177.1.88;
range 192.177.1.120 192.177.1.155;
```
Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.10 - [prefix IP].3.30 dan [prefix IP].3.60 - [prefix IP].3.85 
```
range 192.177.3.10 192.177.3.30;
range 192.177.3.60 192.177.3.85;
```
Client mendapatkan DNS dari WISE dan client dapat terhubung dengan internet melalui DNS tersebut. 
```
option domain-name-servers 192.177.2.2;
```

Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 5 menit sedangkan pada client yang melalui Switch3 selama 10 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 115 menit. 

```
default-lease-time 300;
  max-lease-time 6900;
  
default-lease-time 600;
  max-lease-time 6900;
```

dan diatur interfacenya dengan

```
echo '
INTERFACES="eth0"
' >  /etc/default/isc-dhcp-server
```

atau lengkapnya akan menjadi seperti:

```
subnet 192.177.2.0 netmask 255.255.255.0 {
  option routers 192.177.2.1;
}
subnet 192.177.1.0 netmask 255.255.255.0 {
  range 192.177.1.50 192.177.1.88;
  range 192.177.1.120 192.177.1.155;
  option routers 192.177.1.1;
  option broadcast-address 192.177.1.255;
  option domain-name-servers 192.177.2.2;
  default-lease-time 300;
  max-lease-time 6900;
}
subnet 192.177.3.0 netmask 255.255.255.0 {
  range 192.177.3.10 192.177.3.30;
  range 192.177.3.60 192.177.3.85;
  option routers 192.177.3.1;
  option broadcast-address 192.177.3.255;
  option domain-name-servers 192.177.2.2;
  default-lease-time 600;
  max-lease-time 6900;
} 
  
```

lalu di test pengecekan apakah benar masuk dengan restart salah satu node client,contohnya SSS dan di cek apakah DNS benar masuk dengan `cat /etc/resolv.conf`

![image](https://user-images.githubusercontent.com/92217354/201564952-b2b4bef7-f99a-4c9b-ab9e-e1bb8d41c538.png)

dan di cek menggunakan `ip a`

![image](https://user-images.githubusercontent.com/92217354/201565053-a3396e67-20b6-4d4f-957e-fb0276499938.png)


## No.7
Loid dan Franky berencana menjadikan Eden sebagai server untuk pertukaran informasi dengan alamat IP yang tetap dengan IP [prefix IP].3.13

menambahkan konfigurasi dibawah pada /etc/dhcp/dhcpd.conf

```
host Eden {
    hardware ethernet 7e:fa:3a:a9:b9:e0;
    fixed-address 192.177.3.13;
}
```
hardware internet dapat dilihat pada `ip a` dari Eden

![image](https://user-images.githubusercontent.com/92217354/201565459-3390dfef-be85-4ef3-95d3-c5518fdeb703.png)

setelah itu restart eden dan cek dengan `ip a` apakah benar benar telah fixed address

![image](https://user-images.githubusercontent.com/92217354/201565564-7acf3b3a-022c-41aa-8192-4a8dd693ef0e.png)

