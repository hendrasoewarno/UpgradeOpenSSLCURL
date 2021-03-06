# Upgrade OpenSSL, Curl, PHP dan Apache
Kebanyakan browser besar telah mengakhiri dukungan kepada TLS 1.0 dan TLS 1.1, sehingga kebanyakan webserver didunia telah mengupgrade ke TLS > 1.1. Jika anda menggunakan CURL untuk berkomunikasi dengan server yang telah diupgrade, maka adalah perlu dipastikan bahwa CURL yang digunakan sebagai client juga mendukung TLS > 1.1. Adapun symptom kegagalan koneksi CURL adalah dengan pesan kesalahan:
```
SSL routine: SSL23_GET_SERVER_HELLO:sslv3 alert handshake failure
```

## INSTALASI OPENSSL, CURL, APACHE2, PHP5
--------------------------------------

Pada tulisan ini akan dimulai dengan mengupgrade OpenSSL, CURL, APACHE2 dan PHP5 dan semua dibawah ini adalah menggunakan level power user.
```
sudo su
```

Pastikan bahwa versi Server Ubuntu anda adalah 9.04
```
lsb_release -a
```

Lakukan terlebih dahulu database untuk perintah Locate
```
updatedb
```

File yang dibutuhkan pada proses ini:
1. apache.sh
2. config.nice

Karena Server Ubuntu 9.04 tidak lagi disupport secara resmi, maka kita perlu mengupdate SOURCE.LIST agar dapat melanjutkan proses update dan download package yang dibutuhkan pada aktivitas ini.
```
sed -i 's/us.archive/old-releases/g' /etc/apt/sources.list
sed -i 's/security.ubuntu/old-releases/g' /etc/apt/sources.list
```
perintah tersebut diatas adalah mencari dan mengantikan semua semua string us-archieve.ubuntu.com menjadi old-releases.ubuntu.com pada file /etc/apt/sources.list

Untuk memastikan perubahan tersebut diatas berhasil, maka jalankan anda dapat menjalankan proses update sebagai berikut:
```
apt-get update
```

Karena kita ingin melakukan kompilasi terhadap package OpenSSL, CURL, Apache2 dan PHP5, maka kita perlu menginstalasi semua dependencies yang dibutuhkan:
```
apt-get install build-essential
apt-get build-dep apache2
apt-get build-dep php5
```
dan dependencies untuk openssl
```
apt-get install checkinstall zlib1g-dev
```
da dependecies untuk curl
```
apt-get install -y libssl-dev autoconf libtool make
```
dan dependecies tambahan untuk php5
```
apt-get install libmcrypt-dev
```

## INSTALASI OPENSSL

Periksa versi OpenSSL yang terinstalasi saat ini
```
openssl version
```
Selanjutnya adalah proses download, extract, konfigurasi dan instalasi.
```
cd /usr/local/src
wget https://www.openssl.org/source/openssl-1.0.1o.tar.gz
tar -xf openssl-1.0.1o.tar.gz
cd openssl-1.0.1o

./config --prefix=/usr/local/ssl --openssldir=/usr/local/ssl shared zlib

make
make test
make install
```
Mengaktifkan libssl1.0.1o dengan menambahkan ke ld.so.conf.d
```
echo "/usr/local/ssl/lib" > /etc/ld.so.conf.d/openssl-1.0.1o.conf
```
Perintah tersebut diatas akan membuat file /etc/ld.so.conf.d/openssl-1.0.1o.conf yang berisi /usr/local/ssl/lib untuk referensi ke library baru
Jalankan perintah berikut untuk menghapus library cache dan merefresh kembali
```
rm /etc/ld.so.cache
ldconfig -v
```
Dan pastikan telah ada link ke library yang baru diinstalasi
```
/usr/local/ssl/lib:
	libssl.so.1.0.0 -> libssl.so.1.0.0
	libcrypto.so.1.0.0 -> libcrypti.so.1.0.0
```
Dan tambahkan path eksekusi openssl ke PATH /usr/local/ssl/bin dimana executable OpenSSL berada.
```
sudo pico /etc/environment
PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/usr/local/ssl/bin"
```
Lakukan reload varibel environment
```
source /etc/environment
echo $PATH
```
Jika pada sistem anda telah terinstalasi OpenSSL versi sebelumnya, maka anda perlu disable sekaligus membackup dengan perintah
```
mv /usr/bin/c_rehash /usr/bin/c_rehash.old
mv /usr/bin/openssl /usr/bin/openssl.old
```

## INSTALASI CURL

Periksa versi CURL yang terinstalasi saat ini.
```
curl -V
```
Selanjutnya adalah proses download, extract, konfigurasi dan instalasi.
```
cd /usr/local/src/
```
Download package https://curl.se/download/curl-7.49.1.tar.gz (anda perlu mendownload secara manual pada PC dan menduplikasi melalui USB)
```
tar -xf curl-7.49.1.tar.gz
cd curl-7.49.1
which curl
openssl version -d
```
** catat lokasi OPENSSLDIR
```
./configure -disable-shared -with-ssl=/usr/local/ssl
```
Ingat lokasi -with-ssl=[OPENSSLDIR]
```
make
make install
cp /usr/local/bin/curl /usr/bin/curl
curl -V
```
Pastikan versi CURL adalah 7.49.1 dan telah menggunakan openssl 1.0.1o


## INSTALASI APACHE2

Lakukan backup konfigurasi APACHE2 saat ini (kecuali anda akan kehilangan semua setting Apache2 yang telah ada)
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
** cari dan ganti baris "--with-ssl=/usr/lib/ssl", kita tetap menggunakan libssl0.9.8 yang terinstalasi, karena beberapa dependencies yang kita download adalah membutuhkan/dikompilasi dengan libssl0.9.8
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
** pastikan untuk mengaktifkan module ssl
```
sudo a2enmod ssl
```
Perintah diatas akan membuat symbolic link yang secara manual adalah:
```
#cd /etc/apache2/mods-enabled
#ln -s ../mods-available/ssl.load ssl.load
#ln -s ../mods-available/ssl.conf ssl.conf
```

## INSTALASI PHP

Download https://www.php.net/distributions/php-5.4.45.tar.gz

kalau download pakai curl harus pakai opsi --insecure

```
curl https://www.php.net/distributions/php-5.4.45.tar.gz --insecure
make clean
which curl
```
Buka file config.nice dan koreksi baris '--with-curl=shared,/usr/local/bin/curl' yang merupakan lokasi dimana executable curl berada
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

** karena curl dicompile sebagai module, sehingga perlu diaktifkan sebagai extension dengan membuat curl.ini pada /etc/php5/conf.d
```
locate curl.so
ldd /usr/local/src/20100525/curl.so
```
Pastikan telah menggunakan versi libssl dan libcurl yang sesuai
```
cd /etc/php5/conf.d
pico curl.ini
	; configuration for php CURL module
	extension=curl.so
```

Duplikasi test.php ke folder /var/www untuk menguji perintah curl pada PHP, dengan jalankan
```
wget http://localhost/test.php
```
dan periksa isi file test.php.1 apakah ada pesan error.

# Kesimpulan
Karena pada umumnya package pada Ubuntu 9.04 adalah didasarkan pada libssl0.9.8, maka proses kompilasi Apache2+PHP adalah tetap menggunakan header maupun library libssl0.9.8, tetapi untuk CURL kita perlu menggunakan libssl1.0.1o untuk mendukung TLS > 1.1.
