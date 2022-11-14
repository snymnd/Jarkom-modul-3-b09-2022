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

## No 8 - 12(selesai)
- Buat konfigurasi proxy server pada `berlint` dengan menggunakan `squid` yang telah terinstall.
```bash
# backup file dari /etc/squid/squid.conf ke /etc/squid/squid.conf.bak 
mv /etc/squid/squid.conf /etc/squid/squid.conf.bak 

# tambahkan konfigurasi berikut ke /etc/squid/squid.conf
echo '
http_port 8080
visible_hostname Berlint
' > /etc/squid/squid.conf

#restart dan check status dari squid apakan sudah dapat berjalan
service squid restart
service squid status
```
- Jalankan proxy pada client server dalam hal ini adalah `SSS`  dengan command berkut:
```bash
# Menjalankan proxy pada http dan https (192.177.2.3 adalah ip berlint sebagai proxy server, 8000 adalah port yang digunakan)
export http_proxy="http://192.177.2.3:8080"
export https_proxy="https://192.177.2.3:8080"
```
- Periksa apakah proxy sudah berjalan di client server dengan command berikut  
`` env | grep -i proxy``  
jika sudah berjalan akan ditampilkan proxy yang sedang berjalan sebagai berikut
``
http_proxy=http://192.177.2.3:8080
https_proxy=http://192.177.2.3:8080
``

####  1. Client hanya dapat mengakses internet diluar (selain) hari & jam kerja (senin-jumat 08.00 - 17.00) dan hari libur (dapat mengakses 24 jam penuh)
- Tambahkan konfigurasi berikut pada proxy server yaitu `berlint` pada file `/etc/squid/squid.conf`
```bash 
http_port 8080
visible_hostname berlint

#membuat acl untuk sebagai veriable untuk jam kerja senin-jumat (MTWHF) jam 08:00-17:00
acl AVAILABLE_WORKING time MTWHF 08:00-17:00

# melarang akses internet pada jam kerja
http_a ccess deny AVAILABLE_WORKING
```
#### 2. Adapun pada hari dan jam kerja sesuai nomor (1), client hanya dapat mengakses domain loid-work.com dan franky-work.com (IP tujuan domain dibebaskan) 
- Buat domain loid-work.xom dan francky-work.com terlebih dahulu pada `wise` sebagai DNS server, dengan konfigurasi dalam script file sebagai berikut (pastikan bind9 telah terinstall):
``` bash
echo '
zone "loid-work.com" {
        type master;
        file "/etc/bind/jarkom/loid-work.com";
}; 

zone "franky-work.com" {
        type master;
        file "/etc/bind/jarkom/franky-work.com";
};

' > /etc/bind/named.conf.local

mkdir -p /etc/bind/jarkom

echo '
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     franky-work.com. root.franky-work.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      franky-work.com.
@       IN      A       192.177.2.3     ;IP Berlint 192.177.2.3

' > /etc/bind/jarkom/franky-work.com

echo '
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     loid-work.com. root.loid-work.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      loid-work.com.
@       IN      A       192.177.2.3     ;IP Berlint 192.177.2.3

' > /etc/bind/jarkom/loid-work.com

service bind9 restart
```
- setelah domain terbuat, tambahkan domain loid dan franky-work sebagai acl dnsdomain yanga akan kita allow pada jam kerja dan deny diluar jam kerja. dengan konfigurasi pada `/etc/squid/squid.conf` di `berlint` sehingga konfigurasi menjadi sebagai berkut:
```bash
http_port 8080
visible_hostname berlint

acl AVAILABLE_WORKING time MTWHF 08:00-17:00
acl WORKSITES dstdomain loid-work.com franky-work.com

# membolehkan worksite pada jam kerja
http_access allow WORKSITES AVAILABLE_WORKING
# melarang worksite diluar jam kerja
http_access deny WORKSITES !AVAILABLE_WORKING
http_access deny AVAILABLE_WORKING

# restart squid
# service squid restart
# cek status squid
# service squid status
```
#### 3. Saat akses internet dibuka, client dilarang untuk mengakses web tanpa HTTPS. (Contoh web HTTP: http://example.com)
- Gunakan url_regex untuk melarang semua url yang memiliki pola berupa `diawali dengan "http:" dan "https:"` dengan konfigurasi tambahan pada `squid.conf` pada berlint sebagai berkut:
```bash
http_port 8080
visible_hostname berlint

acl AVAILABLE_WORKING time MTWHF 08:00-17:00
acl WORKSITES dstdomain loid-work.com franky-work.com
#url regex sebagai acl
acl HTTP_REGEX url_regex ^http:.*$
acl HTTPS_REGEX url_regex ^https:.*$

http_access allow WORKSITES AVAILABLE_WORKING
http_access deny WORKSITES !AVAILABLE_WORKING
http_access deny AVAILABLE_WORKING
# menolak akses https pada jam kerja
http_access deny AVAILABLE_WORKING HTTPS_REGEX
# menolak semua akses http pada jam kerja apapun (tidak termasuk worksite yang telah di allow dibagian atas)
http_access deny HTTP_REGEX

# service squid restart
# service squid status
```
#### 4. Agar menghemat penggunaan, akses internet dibatasi dengan kecepatan maksimum 128 Kbps pada setiap host (Kbps = kilobit per second; lakukan pengecekan pada tiap host, ketika 2 host akses internet pada saat bersamaan, keduanya mendapatkan speed maksimal yaitu 128 Kbps)
- Tambahkan bandwidth limit pada konfigurasi `squid.conf` di berlint
```bash
http_port 8080
visible_hostname berlint

acl AVAILABLE_WORKING time MTWHF 08:00-17:00
acl WORKSITES dstdomain loid-work.com franky-work.com
acl HTTP_REGEX url_regex ^http:.*$
acl HTTPS_REGEX url_regex ^https:.*$

http_access allow WORKSITES AVAILABLE_WORKING
http_access deny WORKSITES !AVAILABLE_WORKING
http_access deny AVAILABLE_WORKING
http_access deny AVAILABLE_WORKING HTTPS_REGEX
http_access deny HTTP_REGEX

# konfigurasi limit bandwidth
delay_pools 1
delay_class 1 1
delay_parameters 1 16000/64000

# service squid restart
# service squid status
```

#### 5. Setelah diterapkan, ternyata peraturan nomor (4) mengganggu produktifitas saat hari kerja, dengan demikian pembatasan kecepatan hanya diberlakukan untuk pengaksesan internet pada hari libur
- Tambahkan konfigurasi untuk menentukan bahwa bandwidth limit hanya akan dijalankan pada hari libur yait sabtu dan minggu
```bash
http_port 8080
visible_hostname berlint

acl AVAILABLE_WORKING time MTWHF 08:00-17:00
acl WORKSITES dstdomain loid-work.com franky-work.com
acl HTTP_REGEX url_regex ^http:.*$
acl HTTPS_REGEX url_regex ^https:.*$

http_access allow WORKSITES AVAILABLE_WORKING
http_access deny WORKSITES !AVAILABLE_WORKING
http_access deny AVAILABLE_WORKING
http_access deny AVAILABLE_WORKING HTTPS_REGEX
http_access deny HTTP_REGEX

# acl weekdays sebagi waktu untuk hari minggu (S) dan sabtu (A)
acl WEEKDAYS time SA
delay_pools 1
delay_class 1 1
# melalkukan limit bandwidth hanya pada hari weekdays seperti yang telah didefinisikan pada acl di atas
delay_access 1 allow WEEKDAYS
delay_parameters 1 16000/64000

# service squid restart
# service squid status
```


### testing 8-13
Untuk testing bbandwidth limit menggunakan speedtest, akese deny pada http harus dimatikan lebih dahulu agar speedtest bisa diakses karena speedtest menggungakan akses http, lalu jalankan command pada client yang telah terinstall python sebelumnya
```bash
export PYTHONHTTPSVERIFY=0
```
lalu jalankan commadn `speedtest`
 
#### Senin jam 10:00+
- akses http
![image](https://user-images.githubusercontent.com/70748569/201580057-1715774b-2dfb-4b53-9164-88c5e67451e8.png)

- akses https

![image](https://user-images.githubusercontent.com/70748569/201581509-121b6714-aa5d-47bd-a9c8-5192998116db.png)

- akses loid-work.com dan franky-work (sudah berhasil, namun karena domain tidak mengarah ke webservice apapun maka yang tampil adalah error "service unavaliable - connection failed/refused, karena jika terhalang proxy error yang tampil adalah forbiden - acces denied).

![image](https://user-images.githubusercontent.com/70748569/201580526-b595f5a5-9f67-4ad6-b99c-08bfd01045ce.png)
![image](https://user-images.githubusercontent.com/70748569/201580590-829c7af8-c8aa-433e-b20c-331b817ee4f8.png)

- speedtest

![image](https://user-images.githubusercontent.com/70748569/201582951-8d4afb5d-c19b-4657-8fc6-d835e4d6cbbc.png)

#### Senin jam 20:00+
- akses http

![image](https://user-images.githubusercontent.com/70748569/201580812-eefe1bf6-3de6-4211-aab7-5289da5f598b.png)

- akses https

![image](https://user-images.githubusercontent.com/70748569/201581277-69bc02d8-8908-4f9b-9cb0-c05016d5758d.png)

- akses loid dan franky-work.com (disini bisa dilihat jika tidak bisa diakses dan errornya forbiden -access denied bukan service unavaliable seperti yang berhasil diatas)

![image](https://user-images.githubusercontent.com/70748569/201581357-fbf06c7a-f2ec-481e-a6d2-5c9302959976.png)
![image](https://user-images.githubusercontent.com/70748569/201581994-fdb49667-73e6-4a1b-afb6-ccf72f31a7f6.png)


- speedtest

![image](https://user-images.githubusercontent.com/70748569/201583091-be628c95-2ead-4a3a-ab7d-cc0f652270ec.png)

#### Sabtu jam 10:00
- akses http

![image](https://user-images.githubusercontent.com/70748569/201581607-0f6f54df-d52c-4693-a123-872343ebed59.png)

- akses https

![image](https://user-images.githubusercontent.com/70748569/201582200-2c8c40cc-53c0-40bb-b3be-03464c45b627.png)

- akses loid-work dan franky

![image](https://user-images.githubusercontent.com/70748569/201581921-35c004b2-8f51-4dfb-99b2-9a79929a9b47.png)
![image](https://user-images.githubusercontent.com/70748569/201581982-478b0185-ecef-488f-8330-cd9f9c5afe72.png)

- speedtest

![image](https://user-images.githubusercontent.com/70748569/201583196-d6687c7d-7d90-4da6-bd47-9e5c20d5676d.png)
