# OpenSSL CURL APACHE2 PHP5 + MOD_SECURITY
Proses Upgrade Package OpenSSL, Curl, Apache2 dan libapache-mod-security dan PHP5 pada Server Ubuntu 9.04

## INSTALASI OPENSSL, CURL, APACHE2, PHP5
--------------------------------------

Semua proses dibawah ini adalah menggunakan power user
```
sudo su
```

Pastikan Versi Server Ubuntu anda adalah 9.04
```
lsb_release -a
```

Lakukan update database Locate
```
dbupdate
```

File yang dibutuhkan:
1. apache.sh
2. config.nice

Karena Server Ubuntu 9.04 tidak lagi disupport, sehingga kita perlu mengupdate SOURCE.LIST pada /etc/apt/source.list
```
pico /etc/apt/sources.list
```
cari dan ganti semua semua url us-archieve.ubuntu.com menjadi old-releases.ubuntu.com

Kemudian jalankan proses update SOURCE.LIST
```
apt-get update
```

Jalankan perintah berikut ini untuk menginstalasi semua dependencies yang dibutuhkan untuk proses kompilasi package Apache2 dan PHP2
```
apt-get install build-essential
apt-get build-dep apache2
apt-get build-dep php5
```
atau

** dependencies untuk openssl
```
apt-get install checkinstall zlib1g-dev
```
** dependecies untuk curl
```
apt-get install -y libssl-dev autoconf libtool make
```
** dependecies untuk apache2
```
apt-get install libapr1 libapr1-dev libaprutil1-dev apache2-threaded-dev
```

** dependecies untuk php5
```
apt-get install \
    libxml2-dev \
    libcurl4-openssl-dev \
    libjpeg62-dev \
    libpng12-dev \
    libxpm-dev \
    libmysqlclient15-dev \
    libicu-dev \
    libfreetype6-dev \
    libldap2-dev \
    libxslt-dev \
    libssl-dev \
    libldb-dev \
    libbz2-dev \
    libt1-dev \
    libgmp3-dev \
    libmcrypt-dev \
    libsasl2-dev \
    freetds-dev \
    libpspell-dev \
    librecode-dev \
    libsnmp-dev \
    libtidy-dev
```

## INSTALASI OPENSSL

Periksa versi OpenSSL yang terinstalasi saat ini

openssl version
```
cd /usr/local/src/
```

Download package https://www.openssl.org/source/openssl-1.0.1o.tar.gz
```
tar -xf openssl-1.0.1o.tar.gz
cd openssl-1.0.1o

./config --prefix=/usr --openssldir=/usr/lib/ssl shared zlib

make
make test
make install

which openssl

openssl version -a
```

## INSTALASI CURL

Periksa versi CURL yang terinstalasi saat ini.
```
curl -V

cd /usr/local/src/
```
Download package https://curl.se/download/curl-7.49.1.tar.gz
```
tar -xf curl-7.49.1.tar.gz

cd curl-7.49.1

which curl

openssl version -d
```
** catat lokasi OPENSSLDIR
```
./configure -disable-shared -with-ssl=/usr/lib/ssl
```
Ingat folder harus disesuaikan dengan lokasi [OPENSSLDIR]
```
make

make install

cp /usr/local/bin/curl /usr/bin/curl

curl -V
```
Pastikan CURL adalah telah menggunakan openssl 1.0.1o


## INSTALASI APACHE2

Lakukan backup konfigurasi APACHE2 saat ini
```
sudo cp -r /etc/apache2 ~/apache2_conf_back

sudo su
cd /usr/local/src
curl https://archive.apache.org/dist/httpd/httpd-2.2.31.tar.gz --output httpd-2.2.31.tar.gz
tar -xf httpd-2.2.31.tar.gz
cd httpd-2.2.31

make clean

openssl version -d
```
** catat lokasi OPENSSLDIR
```
sudo pico apache.sh
```
** cari dan ganti baris "--with-ssl=/usr/lib/ssl", harus sama dengan lokasi [OPENSSLDIR]
```
./apache.sh
make

make install

ldd /usr/lib/apache2/modules/mod_ssl.so
```
** pastikan versi libssl sudah sesuai

Kembalikan backup konfigurasi APACHE2 yang telah dibackup sebelumnya
```
sudo rm -rf /etc/apache2
sudo cp -r ~/apache2_conf_back /etc/apache2

apache2 -v
```
** pastikan ssl.conf dan ssl.load sudah dienable pada /etc/apache2/mods-enabled
```
ln -s ../mod-available/ssl.load ssl.load
ln -s ../mod-available/ssl.conf ssl.conf
```
## INSTALASI PHP

Download https://www.php.net/distributions/php-5.4.45.tar.gz

kalau download pakai curl harus pakai opsi --insecure

```
make clean
```
** Buka config.nice dan koreksi baris '--with-curl=shared,/usr/bin/curl' \
```
./config.nice
make install
```
---------------------------------------------Jika terjadi error

  apxs:Error: Activation failed for custom /etc/apache2/httpd.conf
  file..
  apxs:Error: At least one `LoadModule' directive already has to exist..
  make: *** [install-sapi] Error 1

	tambahkan baris dummy ke /etc/apache2/httpd.conf

	#LoadModule dummy_module /usr/lib/apache2/modules/mod_dummy.so

	find /etc/php5/conf.d/ -name "*.ini" -exec sed -i -re 's/^(\s*)#(.*)/\1;\2/g' {} \;

  dan remark beberapa error
  
---------------------------------------------Jika terjadi error
```
libtool --finish /usr/local/src/php-5.4.45/libs
```

** karena curl dicompile sebagai module, sehingga perlu diaktifkan sebagai extension
buat curl.ini pada /etc/php5/conf.d
```
; configuration for php CURL module
extension=curl.so
```

Duplikasi test.php ke folder /var/www, dan jalankan
```
php test.php
```
## INSTALASI MOD-SECURITY
```
apt-get install libapache-mod-security
service apache2 restart
```
