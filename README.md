# Panduan Install Web Server Ubuntu 24.04 (LTS) x64

## Kebutuhan Server

- PHP 8.1 ✅
- Composer ✅
- MySQL ✅
- Apache ✅
- Git ✅

## Langkah-langkah Instalasi

### 1. Update Ubuntu & Install Tools Dasar

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y software-properties-common unzip curl
```

### 2. Install PHP 8.1 dan Ekstensi Laravel

```bash
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update

sudo apt install -y php8.1 php8.1-cli php8.1-mbstring php8.1-xml php8.1-bcmath php8.1-curl php8.1-mysql php8.1-zip php8.1-common php8.1-readline
```

### 3. Install Apache

```bash
sudo apt install -y apache2 libapache2-mod-php8.1
sudo systemctl enable apache2
sudo systemctl start apache2
```

### 4. Install MySQL

```bash
sudo apt install -y mysql-server
sudo systemctl enable mysql
sudo systemctl start mysql

#enable apache agar otomatis jalan saat boot
sudo systemctl enable apache2

# Set password root
sudo mysql_secure_installation
```

> **Catatan Penting**: Saat install Composer tidak bisa menggunakan root user. Gunakan user biasa yang memiliki hak akses sudo.
> Contoh: `usermod -aG sudo (namauser)`

### 5. Install Composer

```bash
cd ~
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
composer --version
```

### 6. Install Git

```bash
sudo apt install -y git
```

## Persiapan Aplikasi Laravel

### 1. Clone Project Laravel dari Git

```bash
cd /var/www/html
sudo git clone https://umarovk:(token))@github.com/umarovk/spmb-sekolah.git spmb
cd spmb
```

> **Catatan**: Format git clone menggunakan token:
> `https://<username>:<token>@github.com/<user>/<repo>.git`

### 2. Set File Permission

```bash
sudo chown -R www-data:www-data /var/www/html/spmb
sudo chmod -R 775 storage
sudo chmod -R 775 bootstrap/cache
```

### 3. Install Dependency dengan Composer

```bash
composer install
```

### 4. Salin File .env dan Generate Key

```bash
cp .env.example .env
php artisan key:generate
```

Edit file `.env` dengan konfigurasi berikut:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=nama_database
DB_USERNAME=root
DB_PASSWORD=yourpassword

App_URL=IP Vps

#Jika tidak pakai https
SESSION_SECURE_COOKIE=false
```

### 5. Set Up Database

```bash
mysql -u root -p
CREATE DATABASE nama_database;
```

### 6. Buat Grant User

```sql
CREATE USER 'admin'@'localhost' IDENTIFIED BY 'passwordku';
GRANT ALL PRIVILEGES ON *.* TO 'admin'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

### 7. Input Migrasi dan Seeder

```bash
php artisan migrate
php artisan db:seed
```

### 8. Set Up Virtual Host (Apache)

Buat file: `/etc/apache2/sites-available/spmb.conf`

```apache
<VirtualHost *:80>
    ServerName spmb.local
    DocumentRoot /var/www/html/spmb/public

    <Directory /var/www/html/spmb/public>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/spmb_error.log
    CustomLog ${APACHE_LOG_DIR}/spmb_access.log combined
</VirtualHost>
```

Aktifkan situs dan mod_rewrite:

```bash
sudo a2ensite spmb
sudo a2enmod rewrite
sudo systemctl restart apache2
```

### Troubleshooting Tambahan

```bash
sudo chown -R www-data:www-data /var/www/html/spmb
sudo chmod -R 755 /var/www/html/spmb
```


## Selesai 🎉

Laravel sekarang seharusnya bisa diakses via browser. 
Langkah selanjutnya bisa meliputi:
- Setup HTTPS/SSL
- Setup Supervisor (untuk queue)
- Setup cronjob untuk scheduler Laravel

Semoga panduan ini membantu diriku dan orang lain yang ingin deploy Laravel tanpa perlu googling lagi dari nol!
