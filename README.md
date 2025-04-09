# Mautic Self-Hosted Installation on DigitalOcean (Ubuntu 22.04)

This guide outlines how to install and configure Mautic (v5+) on a DigitalOcean VPS with Ubuntu 22.04.

---

## ✨ Requirements

- DigitalOcean VPS (2GB RAM minimum)
- Ubuntu 22.04 (fresh install)
- Root access to the server
- Node.js 18+, PHP 8.2+, MariaDB 10.11+
- (Optional) Domain + SMTP credentials

---

## 🚀 Step-by-Step Installation

### 1. Connect to your VPS
```bash
ssh root@your_server_ip
```

### 2. Update the system
```bash
apt update && apt upgrade -y
```

### 3. Install core dependencies
```bash
apt install apache2 unzip curl git software-properties-common -y
```

### 4. Install PHP 8.2 and extensions
```bash
add-apt-repository ppa:ondrej/php -y
apt update
apt install php8.2 php8.2-{cli,mysql,curl,zip,xml,mbstring,imap,bcmath,gd,intl,common,opcache,fpm} libapache2-mod-php8.2 -y
```

### 5. Install Composer
```bash
apt install composer -y
```

### 6. Install Node.js 18 and npm
```bash
curl -fsSL https://deb.nodesource.com/setup_18.x | bash -
apt install -y nodejs
```

### 7. Upgrade MariaDB to 10.11
```bash
systemctl stop mariadb
apt remove mariadb-server mariadb-client -y
curl -LsS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | bash -s -- --mariadb-server-version=10.11
apt update
apt install mariadb-server mariadb-client -y
systemctl start mariadb
```

### 8. Create Mautic Database
```bash
mysql -u root -p
CREATE DATABASE mautic;
CREATE USER 'mauticuser'@'localhost' IDENTIFIED BY 'StrongPassword';
GRANT ALL PRIVILEGES ON mautic.* TO 'mauticuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### 9. Download and Install Mautic
```bash
cd /var/www/html
rm -rf *
git clone https://github.com/mautic/mautic.git .
composer install
npm install
npm run build
```

### 10. Set permissions
```bash
chown -R www-data:www-data /var/www/html
chmod -R 755 /var/www/html
```

### 11. Configure Apache
Edit `/etc/apache2/sites-available/000-default.conf`:
```apache
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html

    <Directory /var/www/html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Enable rewrite module and restart Apache:
```bash
a2enmod rewrite
systemctl restart apache2
```

### 12. Finalize Mautic via Browser
Visit: `http://your_server_ip`
- Fill database info
- Set admin user
- (Optional) Configure SMTP settings

---

## ⏰ Cron Jobs for Mautic
Create a cron file:
```bash
nano /etc/cron.d/mautic
```
Paste:
```cron
*/5 * * * * www-data php /var/www/html/bin/console mautic:segments:update
*/10 * * * * www-data php /var/www/html/bin/console mautic:campaigns:trigger
*/15 * * * * www-data php /var/www/html/bin/console mautic:emails:send
```

---

## 🛡 Security Tip
Delete the test PHP file:
```bash
rm /var/www/html/info.php
```

---

## ✅ Done!
You now have a fully working Mautic installation. You can:
- Create email templates
- Build marketing campaigns
- Track contacts and automate workflows

Need help setting up SMTP (Mailgun, Gmail, SendGrid)? Let us know!

