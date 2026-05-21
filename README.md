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
