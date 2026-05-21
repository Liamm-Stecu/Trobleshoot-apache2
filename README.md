# Cara Import File .sql ke MariaDB Debian

## 1. Pastikan MariaDB Jalan

Cek status:

```bash id="20ckqo"
sudo systemctl status mariadb
```

Kalau belum jalan:

```bash id="04fmrh"
sudo systemctl start mariadb
```

---

## 2. Masuk ke MariaDB

```bash id="qlbr0r"
sudo mysql
```

---

## 3. Buat Database

Contoh:

```sql id="1l9s3l"
CREATE DATABASE ecommerce;
```

Cek database:

```sql id="v9zkaf"
SHOW DATABASES;
```

Keluar:

```sql id="x70szh"
EXIT;
```

---

## 4. Upload File SQL

Contoh lokasi:

```text id="5u3r7v"
/var/www/html/ecommerce/ecommerce.sql
```

---

## 5. Import File SQL

Command:

```bash id="k9q75t"
mysql -u root -p ecommerce < /var/www/html/ecommerce/ecommerce.sql
```

Kalau MariaDB Debian pakai unix_socket:

```bash id="t9b86d"
sudo mysql ecommerce < /var/www/html/ecommerce/ecommerce.sql
```

---

## 6. Kalau Error Saat Import

### Cek Isi SQL

Buka file:

```bash id="0x58t0"
nano /var/www/html/ecommerce/ecommerce.sql
```

Cari:

```text id="f9m6m2"
CREATE TABLE
```

Kalau gak ada:

* SQL export rusak
* atau export cuma data tanpa struktur tabel

---

## 7. Error Umum

### Error:

```text id="e4jlwm"
Table doesn't exist
```

Artinya:

* tabel belum dibuat
* file SQL rusak
* atau export gak lengkap

---

### Error:

```text id="mjlwm6"
Access denied for user
```

Artinya:

* username/password database salah
* atau permission user database belum ada

---

## 8. Export SQL yang Benar dari Windows/XAMPP

Masuk phpMyAdmin:

1. pilih database
2. klik Export
3. pilih Quick
4. format SQL
5. download

---

## 9. Linux Case-Sensitive

Contoh:

```text id="9o1d4k"
Users != users
Product != product
```

Di Windows aman.
Di Linux beda.

---

## 10. Cek Tabel Setelah Import

Masuk MariaDB:

```bash id="4n3l7q"
sudo mysql
```

Pilih database:

```sql id="26j0bq"
USE ecommerce;
```

Lihat tabel:

```sql id="m54r85"
SHOW TABLES;
```

Kalau tabel muncul berarti import berhasil.

---

## 11. Struktur Lokasi Penting

```text id="0xbv48"
/var/www/html        -> file website
/etc/apache2         -> config apache
/var/log/apache2     -> log apache
```

---

## 12. Cek Error Apache/PHP

Kalau web error:

```bash id="blvv6e"
sudo tail -f /var/log/apache2/error.log
```

Itu tempat utama debugging web di Debian.



# Setup Web Ecommerce di Debian + Fix Error Sampai Jalan

## Upload Web

Web diupload ke:

```bash
/var/www/html/ecommerce
```

Biasanya lewat:

* Webmin
* SCP
* WinSCP
* File Manager

---

## Error HTTP 500

Pas buka:

```text
http://IP_DEBIAN
```

muncul:

```text
HTTP ERROR 500
```

Artinya:

* Apache jalan
* tapi backend PHP crash/error

---

## Cek Log Apache

Command:

```bash
sudo tail -f /var/log/apache2/error.log
```

Tujuan:

* lihat error asli PHP/Apache

---

## Error mysqli_connect()

Log:

```text
Call to undefined function mysqli_connect()
```

Artinya:

* PHP MySQL module belum terinstall

---

## Install PHP MySQL Module

Install:

```bash
sudo apt install php-mysql
```

Restart Apache:

```bash
sudo systemctl restart apache2
```

Cek module:

```bash
php -m | grep mysqli
```

Kalau muncul:

```text
mysqli
```

berarti berhasil.

---

## Error Access denied root@localhost

Log:

```text
Access denied for user 'root'@'localhost'
```

Artinya:

* PHP gagal login ke MariaDB
* Debian pake auth unix_socket buat root

Root MariaDB gak bisa dipake langsung dari PHP.

---

## Buat User Database Baru

Masuk MariaDB:

```bash
sudo mysql
```

Buat user:

```sql
CREATE USER 'marcel'@'localhost' IDENTIFIED BY '';
```

Kasih akses:

```sql
GRANT ALL PRIVILEGES ON ecommerce.* TO 'marcel'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

## Edit Config Database PHP

File:

```bash
/var/www/html/ecommerce/config/database.php
```

Dari:

```php
mysqli_connect("localhost","root","","ecommerce");
```

Menjadi:

```php
mysqli_connect("localhost","marcel","password123","ecommerce");
```

---

## Web Tampil Tapi CSS Rusak

Tampilan:

* HTML muncul
* CSS gak kebaca

Penyebab:

* Linux case-sensitive

Contoh:

```text
Css != css
IMG != img
```

Di Windows aman.
Di Linux error.

---

## Fix Path CSS

Cek file CSS:

```bash
find . | grep -i css
```

Cek:

```html
<link rel="stylesheet">
```

Samakan huruf besar kecil path dengan nama folder/file asli.

---

## Fix Permission Web

Kasih permission:

```bash
sudo chown -R www-data:www-data /var/www/html/ecommerce
sudo chmod -R 755 /var/www/html/ecommerce
```

---

## Web Berhasil Jalan

Akhirnya:

* Apache jalan
* PHP jalan
* MariaDB jalan
* CSS jalan
* Ecommerce tampil normal

---

# Pelajaran Penting

## Linux Case-Sensitive

```text
style.css != Style.css
```

---

## HTTP 500 Bukan Error Spesifik

Harus cek:

```bash
/var/log/apache2/error.log
```

---

## PHP Perlu Module Tambahan

Contoh:

```bash
php-mysql
```

---

## Jangan Pakai Root Database Untuk Web Production

Lebih aman:

* buat user database sendiri
* kasih permission seperlunya

---

## Struktur Umum Apache Debian

```text
/var/www/html        -> file web
/etc/apache2         -> config apache
/var/log/apache2     -> log apache
```
