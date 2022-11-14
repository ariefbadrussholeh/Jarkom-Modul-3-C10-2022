# Jarkom-Modul-3-C10-2022

**Laporan Resmi Praktikum Jarkom Modul 3** - _DHCP & Proxy Server_

#### Anggota Kelompok C10:

| Nama                   | NRP        |
| ---------------------- | ---------- |
| Pierra Muhammad Shobr  | 5025201062 |
| Barhan Akmal Falahudin | 5025201008 |
| Arief Badrus Sholeh    | 5025201228 |

## Topologi

Berikut adalah topologi jaringan yang kami buat

![topologi](/screenshot/topologi.png)

> PREFIKS IP -> 192.184

### _Network Configuration_ masing-masing node

Untuk node yang melalui switch 1 dan 3 di*set* `dhcp` sedangkan switch `static`.

- `Ostania`

```
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 192.184.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 192.184.2.1
	netmask 255.255.255.0

auto eth3
iface eth3 inet static
	address 192.184.3.1
	netmask 255.255.255.0
```

- `SSS`

```
auto eth0
iface eth0 inet dhcp
```

- `Garden`

```
auto eth0
iface eth0 inet dhcp
```

- `Wise`

```
auto eth0
iface eth0 inet static
	address 192.184.2.2
	netmask 255.255.255.0
	gateway 192.184.2.1
```

- `Berlint`

```
auto eth0
iface eth0 inet static
	address 192.184.2.4
	netmask 255.255.255.0
	gateway 192.184.2.1
```

- `Westalis`

```
auto eth0
iface eth0 inet static
	address 192.184.2.4
	netmask 255.255.255.0
	gateway 192.184.2.1
```

- `Eden`

```
auto eth0
iface eth0 inet dhcp
```

- `NewstonCastle`

```
auto eth0
iface eth0 inet dhcp
```

- `KemonoPark`

```
auto eth0
iface eth0 inet dhcp
```

## DHCP

Ada beberapa kriteria yang ingin dibuat oleh Loid dan Franky, yaitu:

### Soal 1

> Loid bersama Franky berencana membuat peta tersebut dengan kriteria `WISE` sebagai DNS Server, `Westalis` sebagai DHCP Server, `Berlint` sebagai Proxy Server

#### Konfigurasi pada `Wise`

`Wise` digunakan sebagai DNS Server, sehingga perlu beberapa konfigurasi dilakukan. Pertama lakukan `apt-get update` dan instalasi `bind9`. Kemudian,

- `/etc/bind/named.conf.options`
  > Setting forwarders agar node dapat terhubung ke internet

```bash
options {
    directory "/var/cache/bind";

    forwarders {
        192.168.122.1;
    };

    dnssec-validation auto;
    allow-query(any:);
    auth-nxdomain no;
};
```

- `/etc/bind/named.conf.local`
  > Setting zone untuk dua DNS karena pada bagian PROXY diminta client dapat mengakses `loid-work.com` dan `franky-work.com`

```bash
zone "loid-work.com" (
    type master;
    file "/etc/bind/loid-work/loid-work.com";
);
zone "franky-work.com" (
    type master;
    file "/etc/bind/franky-work/franky-work.com";
);
```

- Membuat direktory

```bash
mkdir /etc/bind/loid-work
mkdir /etc/bind/franky-work
```

- `/etc/bind/loid-work/loid-work.com` dan `/etc/bind/franky-work/franky-work.com`

![network zone](/screenshot/network%20zone.png)

#### Konfigurasi pada `Westalis`

`Wise` digunakan sebagai DHCP Server, sehingga perlu beberapa konfigurasi dilakukan. Pertama lakukan `apt-get update` dan instalasi `isc-dhcp-server`

- `/etc/default/isc-dhcp-server`
  > Menentukan interfaces yang digunakan yaitu `eth0`

```bash
INTERFACES = "eth0"
```

#### Konfigurasi pada `Berlint`

`Berlint` digunakan sebagai Proxy Server. Konfigurasi akan dijelaskan pada bagian [Proxy](#proxy)

### Soal 2

> Ostania sebagai DHCP Relay

#### Konfigurasi pada `Ostania`

`Ostania` digunakan sebagai DHCP Relay, sehingga perlu beberapa konfigurasi dilakukan. Pertama lakukan `apt-get update` dan instalasi `isc-dhcp-relay`

- `/etc/default/isc-dhcp-relay`
  > Memasukkan IP DHCP Server yaitu `192.184.2.4` dan menentukan interface yang digunakan yang melalui setiap switch yaitu `eth1`, `eth2`, dan `eth3`

```bash
SERVERS="192.184.2.4"

INTERFACES="eth1 eth2 eth3"
```

- Start DHCP Relay

```bash
service isc-dhcp-relay restart
```

### Soal 3

> Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server. Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.50 - [prefix IP].1.88 dan [prefix IP].1.120 - [prefix IP].1.155

#### Konfigurasi pada `Westalis`

Agar client pada _switch 1_ mendapatkan konfigurasi IP sesuai yang diminta, perlu dilakukan beberapa konfigurasi

- `/etc/dhcp/dhcpd.conf`
  > Konfigurasi _subnet_ pada _switch 1_. Range dapat dituliskan pada bagian `range "IP_Awal" "IP_Akhir"`

![soal 3](/screenshot/soal%203.png)

### Soal 4

> Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.10 - [prefix IP].3.30 dan [prefix IP].3.60 - [prefix IP].3.85

#### Konfigurasi pada `Westalis`

Agar client pada _switch 3_ mendapatkan konfigurasi IP sesuai yang diminta, perlu dilakukan beberapa konfigurasi

- `/etc/dhcp/dhcpd.conf`
  > Konfigurasi _subnet_ pada _switch 3_. Range dapat dituliskan pada bagian `range "IP_Awal" "IP_Akhir"`

![soal 4](/screenshot/soal%204.png)

### Soal 5

> Client mendapatkan DNS dari WISE dan client dapat terhubung dengan internet melalui DNS tersebut.

#### Konfigurasi pada `Westalis`

Agar client dapat terhubung internet melalui DNS, perlu dilakukan beberapa konfigurasi

- `/etc/dhcp/dhcpd.conf`
  > Menambakan konfigurasi _subnet_ pada masing-masing _switch_. IP DNS Server `192.184.2.2` dapat dituliskan pada bagian `option domain-name-servers "DNS_yang_diinginkan"`

![soal 5](/screenshot/soal%205.png)

### Soal 6

> Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 5 menit sedangkan pada client yang melalui Switch3 selama 10 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 115 menit.

#### Konfigurasi pada `Westalis`

Agar lama waktu DHCP server meminjamkan alamat IP pada client dapat ditentukan, perlu dilakukan beberapa konfigurasi.

- `/etc/dhcp/dhcpd.conf`
  > Menambahkan konfigurasi _subnet_ pada masing-masing _switch_. Waktu peminjaman dapat dituliskan pada bagian `default-lease-time "Waktu"` dan Waktu peminjaman maksimal dapat dituliskan pada bagian `max-lease-time "Waktu"`

![soal 5](/screenshot/soal%206.png)

### Soal 7

> Loid dan Franky berencana menjadikan Eden sebagai server untuk pertukaran informasi dengan alamat IP yang tetap dengan IP [prefix IP].3.13

1. langkah pertama: masuk ke dhcp server di westalis dan masukan code berikut ke /etc/dhcp/dhcpd.conf

```
host Eden{
	hardware ethernet 16:e1:35:75:f0:e4;
	fixed-address 192.184.3.13;
}
```

lalu restart isc-dhcp-server

2. langkah kedua: masuk ke Eden dan masukan code berikut ke /etc/network/interfaces

```
	hwaddress ether 16:e1:35:75:f0:e4
```

lalu restart node Eden pada GNS3

### Testing

- IP masing-masing client
  > Semua IP pada client dipinjamkan dari DHCP Server sesuai dengan range yang telah ditentukan. Termasuk juga _fixed-address_ yang ditetapkan pada `Eden`

![testing-ip](/screenshot/testing-ip.png)

- Ping `google.com`
  > Semua client dapat terhubung ke internet

![testing-ping google](/screenshot/testing-ping%20google.png)

## PROXY

SSS, Garden, dan Eden digunakan sebagai client Proxy agar pertukaran informasi dapat terjamin keamanannya, juga untuk mencegah kebocoran data.

Pada Proxy Server di Berlint, Loid berencana untuk mengatur bagaimana Client dapat mengakses internet. Artinya setiap client harus menggunakan Berlint sebagai HTTP & HTTPS proxy. Adapun kriteria pengaturannya adalah sebagai berikut:
### Proxy Server
Proxy server yang digunakan adalah Squid. Adapun konfigurasi untuk nomor 1 - 5 adalah sebagai berikut:
```sh
apt-get update
apt-get install squid -y
mv /etc/squid/squid.conf /etc/squid/squid.conf.bak
echo "nameserver 192.194.2.2" > /etc/resolve.conf
echo "http_port 8080
visible_hostname Berlint

acl WORKDAYS time MTWHF 08:00-17:00
acl WHITELISTS dstdomain loid-work.com franky-work.com
acl HTTP_REGEX url_regex ^http:.*$

http_access allow WHITELISTS WORKDAYS
http_access deny WORKDAYS
http_access deny HTTP_REGEX

delay_pools 1
delay_class 1 1
delay_access 1 deny WORKDAYS
delay_parameters 1 16000/64000" > /etc/squid/squid.conf
service squid restart
service squid status
```
### Client Proxy
```sh
apt-get update
apt-get install lynx -y
apt-get install speedtest-cli -y
export PYTHONHTTPSVERIFY=0
export http_proxy="http://192.184.2.3:8080"
```

### Soal 1

> Client hanya dapat mengakses internet diluar (selain) hari & jam kerja (senin-jumat 08.00 - 17.00) dan hari libur (dapat mengakses 24 jam penuh)

Untuk dapat melakukan pembatasan, pada konfigurasi proxy server ditambahkan http_access deny pada waktu yang disebutkan. Adapun konfigurasinya adalah sebagai berikut:
```
acl WORKDAYS time MTWHF 08:00-17:00
http_access deny WORKDAYS
```

### Soal 2

> Adapun pada hari dan jam kerja sesuai nomor (1), client hanya dapat mengakses domain loid-work.com dan franky-work.com (IP tujuan domain dibebaskan)

Berdasarkan tabel, diketahui bahwa pada saat jam kerja hanyalah dua domain yang disebutkan yang dapat diakses, dan dua domain tersebut tidak dapat diakses kecuali pada saat di luar jam kerja. Konfigurasi pada squid adalah sebagai berikut:
```
acl WORKDAYS time MTWHF 08:00-17:00
acl WHITELISTS dstdomain loid-work.com franky-work.com
http_access allow WHITELISTS WORKDAYS
```
Adapun makna dari konfigurasi tersebut adalah memberikan akses http pada kedua domain tersebut DAN pada saat jam kerja

### Soal 3

> Saat akses internet dibuka, client dilarang untuk mengakses web tanpa HTTPS. (Contoh web HTTP: http://example.com)

Membatasi https dapat dilakukan dengan beberapa cara, yaitu dengan: seleksi regex dan seleksi port. Untuk kasus ini kami menggunakan seleksi regex. Adapun konfigurasi seleksi regex adalah sebagai berikut:
```
acl HTTP_REGEX url_regex ^http:.*$
http_access deny HTTP_REGEX
```

### Soal 4

> Agar menghemat penggunaan, akses internet dibatasi dengan kecepatan maksimum 128 Kbps pada setiap host (Kbps = kilobit per second; lakukan pengecekan pada tiap host, ketika 2 host akses internet pada saat bersamaan, keduanya mendapatkan speed maksimal yaitu 128 Kbps)

Pembatasan bandwith dapat dilakukan dengan modul delay pada squid. Adapun konfigurasinya adalah sebagai berikut:
```
delay_pools 1
delay_class 1 1
delay_access 1 allow all
delay_parameters 1 16000/64000
```
Adapun penjelasan konfigurasi tersebut adalah sebagai berikut. Pada soal disebutkan bahwa bandwith akan dibagikan secara sama rata pada semua user. Didapatkan informasi bahwa:
1. Hanya terdapat satu jenis user
2. Bandwith dibagi `sama rata` pada semua user
Oleh karena itu, dibuat satu delay pool dengan tipe class 1. Selanjutnya bandwith dibatasi sesuai dengan soal yaitu 128 Kbps.

### Soal 5

> Setelah diterapkan, ternyata peraturan nomor (4) mengganggu produktifitas saat hari kerja, dengan demikian pembatasan kecepatan hanya diberlakukan untuk pengaksesan internet pada hari libur

Pada soal ini, diminta untuk melakukan pembatasan bandwith hanya pada saat selain hari kerja. Oleh karena itu, akses delay pool tidak dberikan kepada semua. Adapun konfigurasi tersebut adalah sebagai berikut:
```
acl WORKDAYS time MTWHF 08:00-17:00
delay_access 1 deny WORKDAYS
```

## Kendala

- Sempat terkendala saat mengkonfigurasi DHCP server dengan relay. DHCP Server failed to start. Solusi meng-copy file konfigurasi asli dari pada menggunakan `echo`
- Terjadi kendala environment `http_proxy` tidak dapat diubah jika ditempatkan di file selain `.bashrc`. Akan tetapi permasalahan dapat diselesaikan dengan menempatkan `export` langsung pada `.bashrc`
