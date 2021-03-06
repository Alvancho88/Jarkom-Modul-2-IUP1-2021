# Jarkom-Modul-2-IUP1-2021

## Group Member Name:
1. (05111942000010) Agustinus Aldi Irawan
2. (05111942000017) Juan Carlos Tepanus Pardosi
3. (05111942000028) Salma Izzatul Islam

## Set The Topology (Edit Network Configuration)

#### Foosha (Router)
``` 
auto eth0
iface eth0 inet dhcp

auto eth1
iface eth1 inet static
	address 10.38.1.1
	netmask 255.255.255.0

auto eth2
iface eth2 inet static
	address 10.38.2.1
	netmask 255.255.255.0 
 ```

#### EniesLobby (DNS Master)
```
auto eth0
iface eth0 inet static
	address 10.38.2.2
	netmask 255.255.255.0
	gateway 10.38.2.1

```


#### Water7 (DNS Slave)
```
auto eth0
iface eth0 inet static
	address 10.38.2.3
	netmask 255.255.255.0
	gateway 10.38.2.1

```


#### Skypie (Web Server)
```
auto eth0
iface eth0 inet static
	address 10.38.2.4
	netmask 255.255.255.0
	gateway 10.38.2.1

```


#### Loguetown (Client)
```
auto eth0
iface eth0 inet static
	address 10.38.1.2
	netmask 255.255.255.0
	gateway 10.38.1.1
```

#### Alabasta (Client)
```
auto eth0
iface eth0 inet static
	address 10.38.1.3
	netmask 255.255.255.0
	gateway 10.38.1.1

```


## no. 1

EniesLobby akan dijadikan sebagai DNS Master, Water7 akan dijadikan DNS Slave, dan Skypie akan digunakan sebagai Web Server. Terdapat 2 Client yaitu Loguetown, dan Alabasta. Semua node terhubung pada router Foosha, sehingga dapat mengakses internet

### Jawab

Menjalankan command `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.38.0.0/16` yang digunakan supaya dapat terhubung ke jaringan luar pada router `Foosha`


Kemudian menambahkan command `iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.38.0.0/16` ke `/root/.bashrc` agar dijalankan setiap kali project distart.


Setelah itu pada semua node lainnya ditambahkan command `echo nameserver 192.168.122.1 > /etc/resolv.conf"` agar dapat terhubung dengan router Foosha dan melakukan hal seperti update dan instalasi


## no. 2

Membuat website utama dengan mengakses franky.yyy.com dengan alias www.franky.yyy.com pada folder kaizoku

### Jawab

pada EniesLobby jalankan command `apt-get update` dan `apt-get install bind9 -y` untuk menginstall bind9


Kemudian mengedit `/etc/bind/named.conf.local` dengan menambahkan :

```bash
    zone "franky.IUP1.com" {
            type master;
            file "/etc/bind/kaizoku/franky.IUP1.com";
    };
```



lalu dibuat folder kaizoku di `/etc/bind`. Lalu copy `/etc/bind/db.local` menjadi `/etc/bind/kaizoku/franky.IUP1.com`. Lalu konfigurasi file tersebut agar memiliki SOA `franky.IUP1.com.`, NS `franky.IUP1.com.`, record A yang mengarah ke `IP Skypie`, dan CNAME `www` pada `franky.IUP1.com.`.


## no. 3

Setelah itu buat subdomain super.franky.yyy.com dengan alias www.super.franky.yyy.com yang diatur DNS nya di EniesLobby dan mengarah ke Skypie.

### Jawab

Mengedit file `/etc/bind/kaizoku/franky.IUP1.com` dengan menambahkan:

```bash
        ns1     IN      A       10.38.2.4 ; IP Skypie
        super   IN      NS      ns1
```


Kemudian menambahkan zone pada `/etc/bind/named.conf.local` dengan menambahkan:

```bash
    zone "super.franky.IUP1.com" {
            type master;
            file "/etc/bind/kaizoku/super.franky.IUP1.com";
    };
```



Lalu copy `/etc/bind/db.local` menjadi `/etc/bind/kaizoku/super.franky.IUP1.com`. Lalu konfigurasi file tersebut agar memiliki SOA `super.franky.IUP1.com.`, NS `super.franky.IUP1.com.`, record A yang mengarah ke `IP Skypie`, dan CNAME `www` pada `super.franky.IUP1.com.`.



## no. 4

Buat juga reverse domain untuk domain utama

### jawab

Pertama, menambahkan zone pada `/etc/bind/named.conf.local` dengan menambahkan:

```bash
    zone "2.38.10.in-addr.arpa" {
            type master;
            file "/etc/bind/kaizoku/2.38.10.in-addr.arpa";
    };
```




Lalu copy `/etc/bind/db.local` menjadi `/etc/bind/kaizoku/2.38.10.in-addr.arpa`. Lalu konfigurasi file tersebut agar memiliki SOA `franky.IUP1.com.`,`2.38.10.in-addr.arpa.` yang memiliki NS `franky.IUP1.com.`, dan `2` yang merupakan byte ke-4 IP EniesLobby memiliki PTR `franky.IUP1.com.`.



## no. 5

Supaya tetap bisa menghubungi Franky jika server EniesLobby rusak, maka buat Water7 sebagai DNS Slave untuk domain utama

### jawab

Di EniesLobby:

Pertama edit zone `franky.IUP1.com` pada `/etc/bind/named.conf.local` menjadi:

```bash
    zone "franky.IUP1.com" {
            type master;
            notify yes;
            also-notify { 10.38.2.3; };
            allow-transfer { 10.38.2.3; };
            file "/etc/bind/kaizoku/franky.IUP1.com";
    };
```



Di Water7:
Jalankan command `apt-get update` dan `apt-get install bind9 -y` untuk menginstall bind9

Kemudian mengedit `/etc/bind/named.conf.local` dengan menambahkan :

```bash
    zone "franky.IUP1.com" {
        type slave;
        masters { 10.38.2.2; };
        file "/var/lib/bind/franky.IUP1.com";
    };
```



## no. 6

Setelah itu terdapat subdomain mecha.franky.yyy.com dengan alias www.mecha.franky.yyy.com yang didelegasikan dari EniesLobby ke Water7 dengan IP menuju ke Skypie dalam folder sunnygo.

### Jawab

Di EniesLobby:

Pertama, mengedit file `/etc/bind/kaizoku/franky.IUP1.com` dengan menambahkan:

```bash
        ns2     IN      A       10.38.2.3 ; IP Water7
        mecha   IN      NS      ns2
```



Kemudian mengedit file `/etc/bind/named.conf.options` dengan comment bagian `dnssec-validation auto;` dan menambahkan line `allow-query{any;};`.



Pastikan sudah ada line `allow-transfer { "IP Water7"; };` pada zone `franky.IUP1.com` di file `/etc/bind/named.conf.local`.



Di Water7:

Pertama edit file `/etc/bind/named.conf.options` dengan comment bagian `dnssec-validation auto;` dan menambahkan line `allow-query{any;};`.



Kemudian menambahkan zone pada `/etc/bind/named.conf.local` dengan menambahkan:

```bash
    zone "mecha.franky.IUP1.com" {
            type master;
            file "/etc/bind/sunnygo/mecha.franky.IUP1.com";
    };
```



lalu dibuat folder sunnygo di `/etc/bind`. Lalu copy `/etc/bind/db.local` menjadi `/etc/bind/sunnygo/mecha.franky.IUP1.com`. Lalu konfigurasi file tersebut agar memiliki SOA `mecha.franky.IUP1.com.`, NS `mecha.franky.IUP1.com.`, record A yang mengarah ke `IP Skypie`, dan CNAME `www` pada `mecha.franky.IUP1.com.`.



## no. 7

Untuk memperlancar komunikasi Luffy dan rekannya, dibuatkan subdomain melalui Franky dengan nama general.mecha.franky.yyy.com dengan alias www.general.mecha.franky.yyy.com yang mengarah ke Skypie.

### Jawab

Di Water7:

Mengedit file `/etc/bind/sunnygo/mecha.franky.IUP1.com` dengan menambahkan:

```bash
        ns1     IN      A       10.38.2.4 ; IP Skypie
        general IN      NS      ns1
```


Kemudian menambahkan zone pada `/etc/bind/named.conf.local` dengan menambahkan:

```bash
    zone "general.mecha.franky.IUP1.com" {
            type master;
            file "/etc/bind/sunnygo/general.mecha.franky.IUP1.com";
    };
```



Lalu copy `/etc/bind/db.local` menjadi `/etc/bind/sunnygo/general.mecha.franky.IUP1.com`. Lalu konfigurasi file tersebut agar memiliki SOA `general.mecha.franky.IUP1.com.`, NS `general.mecha.franky.IUP1.com.`, record A yang mengarah ke `IP Skypie`, dan CNAME `www` pada `general.mecha.franky.IUP1.com.`.



## no. 8

Setelah melakukan konfigurasi server, maka dilakukan konfigurasi Webserver. Pertama dengan webserver www.franky.yyy.com. Pertama, luffy membutuhkan webserver dengan DocumentRoot pada /var/www/franky.yyy.com.

Di Skypie:
Pertama, install `apache2`, `php`, dan `libapache2-mod-php7.0`


Kemudian melakukan `wget` untuk mendownload file yang diperlukan. Setelah itu diunzip sehinggan menampilkan folder-folder seperti ini.



Setelah itu, pindah ke directory `/etc/apache2/sites-available`.Kemudian copy file `000-default.conf` menjadi file `franky.IUP1.com.conf`



Lalu setting file `franky.IUP1.com.conf` agar memiliki line `ServerName franky.IUP1.com`, `ServerAlias www.franky.IUP1.com`, dan `DocumentRoot /var/www/franky.IUP1.com`.



Kemudian bikin directory baru dengan nama `franky.IUP1.com` pada `/var/www/` menggunakan command `mkdir /var/www/franky.IUP1.com`. lalu copy isi dari folder `franky` yang telah didownload ke `/var/www/franky.IUP1.com`.

Setelah itu jalankan command `a2ensite franky.IUP1.com` dan `service apache2 restart`


## no. 9

Setelah itu, Luffy juga membutuhkan agar url www.franky.yyy.com/index.php/home dapat menjadi menjadi www.franky.yyy.com/home.

### Jawab

Pertama, jalankan perintah `a2enmod rewrite` kemudian `service apache2 restart`.

Kemudian pindah ke directory `/var/www/franky.IUP1.com` dan buat file `.htaccess` dengan isi file:

```bash
    RewriteEngine On
    RewriteRule ^home$ index.php/home
```



Kemudian buka file `/etc/apache2/sites-available/franky.IUP1.com.conf` dan tambahkan:

```bash
    <Directory /var/www/franky.IUP1.com>
        Options +FollowSymLinks -Multiviews
        AllowOverride All
    </Directory>
```



## no. 10

Setelah itu, pada subdomain www.super.franky.yyy.com, Luffy membutuhkan penyimpanan aset yang memiliki DocumentRoot pada /var/www/super.franky.yyy.com

### Jawab

Pertama, pindah ke directory `/etc/apache2/sites-available`.Kemudian copy file `000-default.conf` menjadi file `super.franky.IUP1.com.conf`



Lalu setting file `super.franky.IUP1.com.conf` agar memiliki line `ServerName super.franky.IUP1.com`, `ServerAlias www.super.franky.IUP1.com`, dan `DocumentRoot /var/www/super.franky.IUP1.com`.



Kemudian bikin directory baru dengan nama `super.franky.IUP1.com` pada `/var/www/` menggunakan command `mkdir /var/www/super.franky.IUP1.com`. lalu copy isi dari folder `super.franky` yang telah didownload ke `/var/www/super.franky.IUP1.com`.

Setelah itu jalankan command `a2ensite super.franky.IUP1.com` dan `service apache2 restart`


## no. 11

Akan tetapi, pada folder /public, Luffy ingin hanya dapat melakukan directory listing saja.

### Jawab

Pindah ke directory `/etc/apache2/sites-available` kemudian buka file `super.franky.IUP1.com` dan tambahkan:

```bash
    <Directory /var/www/super.franky.IUP1.com/public>
        Options +Indexes
    </Directory>
```



## no. 12

Tidak hanya itu, Luffy juga menyiapkan error file 404.html pada folder /errors untuk mengganti error kode pada apache .

### Jawab

Pindah ke directory `/etc/apache2/sites-available` kemudian buka file `super.franky.IUP1.com` dan tambahkan:

```bash
        ErrorDocument 404 /error/404.html
```



## no. 13

Luffy juga meminta Nami untuk dibuatkan konfigurasi virtual host. Virtual host ini bertujuan untuk dapat mengakses file asset www.super.franky.yyy.com/public/js menjadi www.super.franky.yyy.com/js.

### Jawab

Pindah ke directory `/etc/apache2/sites-available` kemudian buka file `super.franky.IUP1.com` dan tambahkan:

```bash
        Alias "/js" "/var/www/super.franky.IUP1.com/public/js"
```

## no. 14

Dan Luffy meminta untuk web www.general.mecha.franky.yyy.com hanya bisa diakses dengan port 15000 dan port 15500.

### Jawab

Pertama, pindah ke directory `/etc/apache2/sites-available`.Kemudian copy file `000-default.conf` menjadi file `general.mecha.franky.IUP1.com.conf`



Lalu setting file `general.mecha.franky.IUP1.com.conf` agar memiliki `<VirtualHost *:15000 *:15500>`, line `ServerName general.mecha.franky.IUP1.com`, `ServerAlias www.general.mecha.franky.IUP1.com`, dan `DocumentRoot /var/www/general.mecha.franky.IUP1.com`.



Kemudian tambahkan port 15000 dan 15500 pada file `/etc/apache2/ports.conf` dengan menambahkan:

```bash
    Listen 15000
    Listen 15500
```



Kemudian bikin directory baru dengan nama `general.mecha.franky.IUP1.com` pada `/var/www/` menggunakan command `mkdir /var/www/general.mecha.franky.IUP1.com`. lalu copy isi dari folder `general.mecha.franky` yang telah didownload ke `/var/www/general.mecha.franky.IUP1.com`.

Setelah itu jalankan command `a2ensite general.mecha.franky.IUP1.com` dan `service apache2 restart`


## no. 15

dengan authentikasi username luffy dan password onepiece dan file di /var/www/general.mecha.franky.yyy

### Jawab

Pertama, jalankan command `htpasswd -c /etc/apache2/.htpasswd luffy` untuk membuat file yang menyimpan username dan password kedalam file `/etc/apache2/.htpasswd` dengan user `luffy`, lalu akan ada prompt untuk memasukkan dan mengkonfirmasi password.


Kemudian, edit file `/etc/apache2/sites-available/general.mecha.franky.IUP1.com.conf` dengan menambahkan:

```bash
    <Directory /var/www/general.mecha.franky.IUP1.com>
        Options +FollowSymLinks -Multiviews
        AllowOverride All
    </Directory>
```



Lalu, edit file `/var/www/general.mecha.franky.IUP1.com/.htaccess` menjadi:

```bash
    AuthType Basic
    AuthName "Restricted Content"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
```


## no. 16

Dan setiap kali mengakses IP EniesLobby akan diahlikan secara otomatis ke www.franky.yyy.com

### Jawab

Problem : Karena tidak bisa mengakses IP EniesLobby, maka IP yang kami alihkan adalah ketika mengakses IP Skypie
Edit file `/var/www/html/.htaccess` dengan menambahkan:

```bash
    RewriteEngine On
    RewriteBase /
    RewriteCond %{HTTP_HOST} ^10\.38\.2\.4$
    RewriteRule ^(.*)$ http://www.franky.IUP1.com [L,R=301]
```



Kemudian, edit file `/etc/apache2/sites-available/000-default.conf` dengan menambahkan:

```bash
    <Directory /var/www/html>
        Options +FollowSymLinks -Multiviews
        AllowOverride All
    </Directory>
```



## no. 17

Dikarenakan Franky juga ingin mengajak temannya untuk dapat menghubunginya melalui website www.super.franky.yyy.com, dan dikarenakan pengunjung web server pasti akan bingung dengan randomnya images yang ada, maka Franky juga meminta untuk mengganti request gambar yang memiliki substring ???franky??? akan diarahkan menuju franky.png.

### Jawab

Pertama, edit file `/etc/apache2/sites-available/super.franky.IUP1.com.conf` dengan menambahkan:

```bash
    <Directory /var/www/super.franky.IUP1.com>
        Options +FollowSymLinks -Multiviews
        AllowOverride All
    </Directory>
```



Kemudian, edit file `/var/www/super.franky.IUP1.com/.htaccess` dengan menambahkan:

```bash
    RewriteEngine On
    RewriteRule ^(.*)franky(.*)\.(jpg|gif|png)$ http://super.franky.IUP1.com/public/images/franky.png [L,R]
```


# Konfigurasi Full 1-17:
## All

### ./.bashrc
```
/root/config.sh
```

### Terminal
```
chmod +x config.sh

EniesLobby & Water7
service bind9 restart
service bind9 stop

Skypie
service apache2 restart

ps aux | grep ping
kill -9 [pid]
```

## Foosha (Router)
### ./.bashrc
```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE -s 10.38.0.0/16
```

## EniesLobby (DNS Master)

### config.sh
```
# !/bin/sh

echo nameserver 192.168.122.1 > /etc/resolv.conf

apt-get update
apt-get install bind9 -y

mkdir /etc/bind/kaizoku

cp /root/named.conf.local /etc/bind
cp /root/franky.IUP1.com /etc/bind/kaizoku
cp /root/super.franky.IUP1.com /etc/bind/kaizoku
cp /root/2.38.10.in-addr.arpa /etc/bind/kaizoku
cp /root/named.conf.options /etc/bind
```

### named.conf.local
```
zone "franky.IUP1.com" {
        type master;
        notify yes;
        also-notify { 10.38.2.3; };
        allow-transfer { 10.38.2.3; };
        file "/etc/bind/kaizoku/franky.IUP1.com";
};

zone "super.franky.IUP1.com" {
        type master;
        file "/etc/bind/kaizoku/super.franky.IUP1.com";
};

zone "2.38.10.in-addr.arpa" {
        type master;
        file "/etc/bind/kaizoku/2.38.10.in-addr.arpa";
};
```

### named.conf.options
```
        //dnssec-validation auto;
        allow-query{any;};
```

### franky.IUP1.com
```
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     franky.IUP1.com. root.franky.IUP1.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      franky.IUP1.com.
@       IN      A       10.38.2.4 ; IP Skypie
www     IN      CNAME   franky.IUP1.com.
ns1     IN      A       10.38.2.4 ; IP Skypie
super   IN      NS      ns1
ns2     IN      A       10.38.2.3 ; IP Water7
mecha   IN      NS      ns2
```

### super.franky.IUP1.com
```
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     super.franky.IUP1.com. root.super.franky.IUP1.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      super.franky.IUP1.com.
@       IN      A       10.38.2.4
www     IN      CNAME   super.franky.IUP1.com.
```

### 2.38.10.in-addr.arpa
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     franky.IUP1.com. root.franky.IUP1.com. (
                2021100401              ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
2.38.10.in-addr.arpa.   IN      NS      franky.IUP1.com.
2       IN      PTR     franky.IUP1.com.
```

## Water7 (DNS Slave)

### config.sh
```
#!/bin/sh

echo nameserver 192.168.122.1 > /etc/resolv.conf

apt-get update
apt-get install bind9 -y

cp /root/named.conf.local /etc/bind
cp /root/named.conf.options /etc/bind

mkdir /etc/bind/sunnygo

cp /root/mecha.franky.IUP1.com /etc/bind/sunnygo
cp /root/general.mecha.franky.IUP1.com /etc/bind/sunnygo
```

### named.conf.local
```
zone "franky.IUP1.com" {
        type slave;
        masters { 10.38.2.2; };
        file "/var/lib/bind/franky.IUP1.com";
};

zone "mecha.franky.IUP1.com" {
            type master;
            file "/etc/bind/sunnygo/mecha.franky.IUP1.com";
};

zone "general.mecha.franky.IUP1.com" {
        type master;
        file "/etc/bind/sunnygo/general.mecha.franky.IUP1.com";
};
```

### named.conf.options
```
        //dnssec-validation auto;
        allow-query{any;};
```

### mecha.franky.IUP1.com
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     mecha.franky.IUP1.com. root.mecha.franky.IUP1.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      mecha.franky.IUP1.com.
@       IN      A       10.38.2.4
www     IN      CNAME   mecha.franky.IUP1.com.
ns1     IN      A       10.38.2.4 ; IP Skypie
general IN      NS      ns1
```

### general.mecha.franky.IUP1.com
```
;
; BIND data file for local loopback interface
;
$TTL    604800
@       IN      SOA     general.mecha.franky.IUP1.com. root.general.mecha.frank$
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      general.mecha.franky.IUP1.com.
@       IN      A       10.38.2.4
www     IN      CNAME   general.mecha.franky.IUP1.com.
```

## Skypie (Web Server)

### config.sh
```
#!/bin/sh

echo nameserver 192.168.122.1 > /etc/resolv.conf

apt-get update
apt-get install apache2 -y
apt-get install php -y
apt-get install libapache2-mod-php7.0 -y
apt-get install wget -y
apt-get install unzip -y
apt-get install lynx -y

cp franky.IUP1.com.conf /etc/apache2/sites-available

mkdir /var/www/franky.IUP1.com

cp /root/franky/home.html /var/www/franky.IUP1.com
cp /root/franky/index.php /var/www/franky.IUP1.com

a2ensite franky.IUP1.com
service apache2 restart

a2enmod rewrite
service apache2 restart

mkdir /var/www/super.franky.IUP1.com

cp /root/.htaccess /var/www/franky.IUP1.com
cp super.franky.IUP1.com.conf /etc/apache2/sites-available
cp super.franky.IUP1.com /var/www

mkdir /var/www/super.franky.IUP1.com/error

mkdir /var/www/super.franky.IUP1.com/public
mkdir /var/www/super.franky.IUP1.com/public/css
mkdir /var/www/super.franky.IUP1.com/public/images
mkdir /var/www/super.franky.IUP1.com/public/js

cp /root/super.franky/error/404.html /var/www/super.franky.IUP1.com/error

cp /root/super.franky/public/css/bro.css /var/www/super.franky.IUP1.com/public/css
cp /root/super.franky/public/css/edit.css /var/www/super.franky.IUP1.com/public/css
cp /root/super.franky/public/css/main.css /var/www/super.franky.IUP1.com/public/css

cp /root/super.franky/public/images/background-frank.jpg /var/www/super.franky.IUP1.com/public/images
cp /root/super.franky/public/images/bukanfrankytapirandom.99689 /var/www/super.franky.IUP1.com/public/images
cp /root/super.franky/public/images/car.jpg /var/www/super.franky.IUP1.com/public/images
cp /root/super.franky/public/images/eyeoffranky.jpg /var/www/super.franky.IUP1.com/public/images
cp /root/super.franky/public/images/fake-franky.jpg234 /var/www/super.franky.IUP1.com/public/images
cp /root/super.franky/public/images/franky.png /var/www/super.franky.IUP1.com/public/images
cp /root/super.franky/public/images/frankysupersecretfood.cursed /var/www/super.franky.IUP1.com/public/images
cp /root/super.franky/public/images/not-franky.jpg /var/www/super.franky.IUP1.com/public/images
cp /root/super.franky/public/images/papa.franky.jpg /var/www/super.franky.IUP1.com/public/images
cp /root/super.franky/public/images/roboto-frankyasdjklkljqwennmn.listingbro /var/www/super.franky.IUP1.com/public/images
cp /root/super.franky/public/images/stockphotorandomfran.woi /var/www/super.franky.IUP1.com/public/images
cp /root/super.franky/public/images/whatMatters.jpg /var/www/super.franky.IUP1.com/public/images

cp /root/super.franky/public/js/autocomplete.js /var/www/super.franky.IUP1.com/public/js
cp /root/super.franky/public/js/js.js /var/www/super.franky.IUP1.com/public/js
cp /root/super.franky/public/js/reddit.js /var/www/super.franky.IUP1.com/public/js

a2ensite super.franky.IUP1.com
service apache2 restart

cp general.mecha.franky.IUP1.com.conf /etc/apache2/sites-available
cp ports.conf /etc/apache2

mkdir /var/www/general.mecha.franky.IUP1.com

cp /root/general.mecha.franky/drag.wav /var/www/general.mecha.franky.IUP1.com
cp /root/general.mecha.franky/duck-duck-go.jpg /var/www/general.mecha.franky.IUP1.com
cp /root/general.mecha.franky/f1.png /var/www/general.mecha.franky.IUP1.com
cp /root/general.mecha.franky/funko-pop.jpg /var/www/general.mecha.franky.IUP1.com

a2ensite general.mecha.franky.IUP1.com
service apache2 restart

cp 000-default.conf /etc/apache2/sites-available
```

### franky.IUP1.com.conf
```
<VirtualHost *:80>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/franky.IUP1.com
        ServerName franky.IUP1.com
        ServerAlias www.franky.IUP1.com

        <Directory /var/www/franky.IUP1.com>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

### super.franky.IUP1.com.conf
```
<VirtualHost *:80>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/super.franky.IUP1.com
        ServerName super.franky.IUP1.com
        ServerAlias www.super.franky.IUP1.com

        <Directory /var/www/super.franky.IUP1.com/public>
                Options +Indexes
        </Directory>

        <Directory /var/www/super.franky.IUP1.com>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>

        ErrorDocument 404 /error/404.html

        Alias "/js" "/var/www/super.franky.IUP1.com/public/js"

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

### general.mecha.franky.IUP1.com.conf
```
<VirtualHost *:15000 *:15500>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/general.mecha.franky.IUP1.com
        ServerName general.mecha.franky.IUP1.com
        ServerAlias www.general.mecha.franky.IUP1.com

        <Directory /var/www/general.mecha.franky.IUP1.com>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

### ports.conf
```
# If you just change the port or add more ports here, you will likely also
# have to change the VirtualHost statement in
# /etc/apache2/sites-enabled/000-default.conf

Listen 80
Listen 15000
Listen 15500

<IfModule ssl_module>
        Listen 443
</IfModule>

<IfModule mod_gnutls.c>
        Listen 443
</IfModule>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

### 000-default.conf
```
<VirtualHost *:80>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
        <Directory /var/www/html>
                Options +FollowSymLinks -Multiviews
                AllowOverride All
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

### nano /var/www/franky.IUP1.com/.htaccess
```
    RewriteEngine On
    RewriteRule ^home$ index.php/home
```

### nano /var/www/general.mecha.franky.IUP1.com/.htaccess
```
    AuthType Basic
    AuthName "Restricted Content"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user
```

### nano /var/www/html/.htaccess
```
    RewriteEngine On
    RewriteBase /
    RewriteCond %{HTTP_HOST} ^10\.38\.2\.4$
    RewriteRule ^(.*)$ http://www.franky.IUP1.com [L,R=301]
```

### nano /var/www/super.franky.IUP1.com/.htaccess
```
    RewriteEngine On
    RewriteRule ^(.*)franky(.*)\.(jpg|gif|png)$ http://super.franky.IUP1.com/public/images/franky.png [L,R]
```

## LogueTown (Client 1)

### config.sh
```
#!/bin/sh

echo nameserver 192.168.122.1 > /etc/resolv.conf

apt-get update
apt-get install lynx -y
apt-get install dnsutils -y
cp /root/resolv.conf /etc
```

### resolv.conf
```
#nameserver 192.168.122.1
nameserver 10.38.2.2
nameserver 10.38.2.3
```

## Alabasta (Client 2)

### config.sh
```
#!/bin/sh

echo nameserver 192.168.122.1 > /etc/resolv.conf

apt-get update
apt-get install lynx -y
apt-get install dnsutils -y
cp /root/resolv.conf /etc
```

### resolv.conf
```
#nameserver 192.168.122.1
nameserver 10.38.2.2
nameserver 10.38.2.3
```

### Testing
```
1
EniesLobby & Water7
service bind9 restart

Skypie
service apache2 restart

ps aux | grep ping
kill -9 [pid]

2 
nameserver pada etc resolv
ping franky.IUP1.com
ping www.franky.IUP1.com

3 
ping super.franky.IUP1.com
ping www.super.franky.IUP1.com

4 
mv 2.10.38.in-addr.arpa 2.38.10.in-addr.arpa
host -t PTR 10.38.2.2

5
EniesLobby
service bind9 stop

Water7
service bind9 restart

ping franky.IUP1.com

6
ping mecha.franky.IUP1.com
ping www.mecha.franky.IUP1.com

7
ping general.mecha.franky.IUP1.com
ping www.general.mecha.franky.IUP1.com

8
lynx franky.IUP1.com

9
lynx www.franky.IUP1.com/home

nano /var/www/franky.IUP1.com/.htaccess
    RewriteEngine On
    RewriteRule ^home$ index.php/home

10, 11, 12
lynx www.super.franky.IUP1.com
11 listing (public)
12 error

13
lynx www.super.franky.IUP1.com/js

14
lynx www.general.mecha.franky.IUP1.com (error)
lynx http://10.38.2.4:15000

15
lynx http://10.38.2.4:15000

htpasswd -c /etc/apache2/.htpasswd luffy
Masukkan password
nano /var/www/general.mecha.franky.IUP1.com/.htaccess
    AuthType Basic
    AuthName "Restricted Content"
    AuthUserFile /etc/apache2/.htpasswd
    Require valid-user

16
lynx 10.38.2.4
lynx franky.IUP1.com

nano /var/www/html/.htaccess

    RewriteEngine On
    RewriteBase /
    RewriteCond %{HTTP_HOST} ^10\.38\.2\.4$
    RewriteRule ^(.*)$ http://www.franky.IUP1.com [L,R=301]

17
nano /etc/apache2/sites-available/super.franky.IUP1.com.conf

nano /var/www/super.franky.IUP1.com/.htaccess

    RewriteEngine On
    RewriteRule ^(.*)franky(.*)\.(jpg|gif|png)$ http://super.franky.IUP1.com/public/images/franky.png [L,R]

lynx www.super.franky.IUP1.com/images/franky
lynx www.super.franky.IUP1.com/images/franky.png
lynx www.super.franky.IUP1.com/images/franky.jpg
lynx www.super.franky.IUP1.com/images/not-franky.jpg
```

# Screenshots

## Settingan Node

### Foosha

![Screenshot (8947)](https://user-images.githubusercontent.com/61174498/138887114-944d1997-52e8-467a-8788-f0a19a9b3d32.png)

### EniesLobby

![Screenshot (8948)](https://user-images.githubusercontent.com/61174498/138887206-5d17778a-2915-4125-98d8-0f9802adee69.png)

### Water7

![Screenshot (8949)](https://user-images.githubusercontent.com/61174498/138887240-07fff8a1-4c11-4910-ae30-1a932a0c5e1e.png)

### Skypie

![Screenshot (8950)](https://user-images.githubusercontent.com/61174498/138887270-98c956e7-e96c-459f-993a-faacf0350446.png)

### LogueTown

![Screenshot (8951)](https://user-images.githubusercontent.com/61174498/138887292-78647346-5fbc-45ba-841d-16a02d1e8bde.png)

### Alabasta

![Screenshot (8952)](https://user-images.githubusercontent.com/61174498/138887380-6fd94f23-940d-4cad-9b89-b2750cafbd44.png)

## Pengerjaan Soal

### 2

![Screenshot (8995)](https://user-images.githubusercontent.com/61174498/138888229-980f3eb7-beb9-40a9-906a-56b1e1f0c76c.png)

### 3

![Screenshot (8996)](https://user-images.githubusercontent.com/61174498/138888320-facb977a-2f78-4d74-8395-70ef34ea441f.png)

### 4

![Screenshot (8998)](https://user-images.githubusercontent.com/61174498/138888373-417f7645-8410-4592-82e6-f1be2bb5cb9f.png)

### 5

![Screenshot (8999)](https://user-images.githubusercontent.com/61174498/138888436-2b93a635-3d6b-419a-920e-2b8994240981.png)

### 6

![Screenshot (9000)](https://user-images.githubusercontent.com/61174498/138888612-415b82c0-1e72-4d3e-81d6-b10da8248733.png)

### 7

![Screenshot (9001)](https://user-images.githubusercontent.com/61174498/138888649-41e4a730-5aa7-4d87-b83c-da6f7124c9b9.png)

### 8

![Screenshot (9002)](https://user-images.githubusercontent.com/61174498/138888682-427eb161-02f9-40ea-85e8-68de2be0f4fc.png)

### 9

![Screenshot (9004)](https://user-images.githubusercontent.com/61174498/138888826-7f3dda4e-8606-45d9-ad85-b02d80a50cd7.png)

### 10

![Screenshot (9005)](https://user-images.githubusercontent.com/61174498/138888960-14085653-64e6-40fa-9684-b761417a3b5d.png)

### 11

![Screenshot (9007)](https://user-images.githubusercontent.com/61174498/138889082-21f6ec1d-cbd1-44c2-b894-b446799000c9.png)

![Screenshot (9008)](https://user-images.githubusercontent.com/61174498/138889114-279f78c4-2974-4a39-a8bd-fc1e73ea865c.png)

![Screenshot (9011)](https://user-images.githubusercontent.com/61174498/138889146-ad8ca16c-5386-42e1-bf40-e8670db119b4.png)

![Screenshot (9012)](https://user-images.githubusercontent.com/61174498/138889166-d9195167-8485-4163-ae70-32cdc4ff77f9.png)

### 12

![Screenshot (9006)](https://user-images.githubusercontent.com/61174498/138889206-6fc92ef2-458c-40c5-923a-1be9fe61127d.png)

![Screenshot (9013)](https://user-images.githubusercontent.com/61174498/138889234-570c658a-3695-4519-9dc7-d4450da6c805.png)

### 13

![Screenshot (9014)](https://user-images.githubusercontent.com/61174498/138889304-695e086d-cc50-4ec5-bb65-40d4f9c67e86.png)

### 14

![Screenshot (9016)](https://user-images.githubusercontent.com/61174498/138889380-d0737ed8-c7c4-4cab-bd3e-ca1c454b201c.png)

![Screenshot (9017)](https://user-images.githubusercontent.com/61174498/138889426-873121fd-5401-4c77-bc02-e17d701fa4c8.png)

### 15

![Screenshot (9019)](https://user-images.githubusercontent.com/61174498/138889497-498f9c02-fb30-475c-9fb1-bfcefa69d482.png)

![Screenshot (9021)](https://user-images.githubusercontent.com/61174498/138889515-a3d31f9c-3f99-479f-8f88-29cb957d068a.png)

### 16

![Screenshot (9024)](https://user-images.githubusercontent.com/61174498/138889579-aa89c08a-20a9-4c02-985f-739889c6f898.png)

### 17

![Screenshot (9027)](https://user-images.githubusercontent.com/61174498/138889745-e95317d8-6c85-4387-92ef-3f4284de75b0.png)

![Screenshot (9026)](https://user-images.githubusercontent.com/61174498/138889761-9f4e40c4-aa66-4d46-91c5-634e777f5524.png)

# Problems:

a.) Case Sensitive
b.) Pointing to the wrong Node
c.) Backup costs a lot of time
d.) Setting Up costs a lot of time
e.) Do Lynx Display Images?
