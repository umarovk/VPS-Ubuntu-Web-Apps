# Panduan Deploy Laravel di Ubuntu (PHP 8.1)

Dokumen ini dibuat sebagai panduan untuk saya (atau siapa pun) yang ingin melakukan deploy aplikasi Laravel dari awal di server Ubuntu. 
Panduan ini mengasumsikan bahwa kamu menggunakan Laravel versi 10 dengan PHP 8.1 dan server berbasis Ubuntu (20.04 atau lebih baru).

---

## 1. Persiapan Server

### Update & Install Paket Dasar
```bash
sudo apt update
sudo apt install apache2 php8.1 php8.1-cli php8.1-mbstring php8.1-xml php8.1-curl php8.1-mysql unzip curl git composer -y
```

> Tips: Jika menggunakan MySQL, install juga `mysql-server`. Jika menggunakan database lain (PostgreSQL, SQLite), sesuaikan.

### Start MySQL (Jika digunakan)
```bash
sudo systemctl start mysql
sudo systemctl enable mysql
```

---

## 2. Clone Aplikasi Laravel dari Git

Pindah ke direktori web:
```bash
cd /var/www
sudo git clone https://github.com/username/nama-repo.git laravelapp
cd laravelapp
```

Ganti `username/nama-repo.git` sesuai dengan repositori kamu.

---

## 3. Set Hak Akses

Laravel butuh hak akses yang benar agar bisa menulis ke direktori `storage` dan `bootstrap/cache`:

```bash
sudo chown -R www-data:www-data /var/www/laravelapp
sudo chmod -R 755 /var/www/laravelapp
```

---

## 4. Install Dependency Laravel

```bash
composer install
cp .env.example .env
php artisan key:generate
```

> Jangan lupa: Konfigurasi file .env sesuai kebutuhan (DB connection, APP_NAME, APP_URL, dsb).

---

## 5. Konfigurasi Apache

### Buat Virtual Host

```bash
sudo nano /etc/apache2/sites-available/konfig-kamu.conf
```

Isi file:
```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/laravelapp/public

    <Directory /var/www/laravelapp/public>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/spmb_error.log
    CustomLog ${APACHE_LOG_DIR}/spmb_access.log combined
</VirtualHost>
```

> Penting: `DocumentRoot` harus mengarah ke direktori public Laravel.

### Aktifkan Virtual Host dan Modul

```bash
sudo a2ensite konfig-kamu.conf
sudo a2dissite 000-default.conf
sudo a2enmod rewrite
sudo systemctl reload apache2
```

---

## 6. Cek dan Jalankan Apache

Pastikan Apache berjalan:
```bash
sudo systemctl status apache2
```

Jika belum aktif:
```bash
sudo systemctl start apache2
```

---

## 7. Tes Akses Aplikasi

- Akses dari browser: `http://<IP-Server>`
- Jika gagal, coba dari dalam server:
```bash
curl http://localhost
```

Jika curl berhasil tapi akses dari luar tidak bisa:
- Periksa firewall atau security group dari penyedia server cloud kamu (Alibaba, AWS, dll).
- Pastikan port 80 dan 443 dibuka untuk publik.

---

## Troubleshooting Umum

| Masalah                        | Penyebab                     | Solusi                                                                 |
|-------------------------------|------------------------------|------------------------------------------------------------------------|
| ERR_CONNECTION_REFUSED        | Apache tidak aktif / port 80 tertutup | Jalankan Apache, cek security group VPS/Cloud                          |
| 403 Forbidden                 | Izin folder salah            | Cek kepemilikan folder (www-data) dan arahkan DocumentRoot ke /public |
| 500 Internal Server Error     | Masalah permission atau konfigurasi .env | Pastikan storage & logs bisa ditulis, cek .env                        |

---

## Selesai 🎉

Laravel sekarang seharusnya bisa diakses via browser. 
Langkah selanjutnya bisa meliputi:
- Setup HTTPS/SSL
- Setup Supervisor (untuk queue)
- Setup cronjob untuk scheduler Laravel

Semoga panduan ini membantu diriku dan orang lain yang ingin deploy Laravel tanpa perlu googling lagi dari nol!
