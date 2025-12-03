# 🚀 ArrowPulse - Production Installation Guide

**Complete Installation Guide for Production Deployment**

This guide walks you through installing ArrowPulse on a fresh server from scratch. Follow these steps carefully to ensure a successful production deployment.

---

## 📋 Table of Contents

1. [Overview](#overview)
2. [System Requirements](#system-requirements)
3. [Server Preparation](#server-preparation)
4. [Installation Steps](#installation-steps)
5. [Setup Wizard](#setup-wizard)
6. [Web Server Configuration](#web-server-configuration)
7. [Queue Workers Setup](#queue-workers-setup)
8. [Post-Installation](#post-installation)
9. [Verification](#verification)
10. [Troubleshooting](#troubleshooting)

---

## 📖 Overview

ArrowPulse is a comprehensive monitoring and analytics dashboard for GreenArrow MTA. This guide covers:

- ✅ Complete server setup from scratch
- ✅ Automated installation process
- ✅ Interactive setup wizard
- ✅ Production configuration
- ✅ Web server setup (Apache/Nginx)
- ✅ Background job processing
- ✅ Security best practices

**Estimated Installation Time:** 30-45 minutes

---

## 🔧 System Requirements

### **Minimum Server Specifications**

- **CPU:** 2 cores
- **RAM:** 4 GB
- **Storage:** 20 GB free space
- **Network:** Internet connection for license activation and optional AI features

### **Software Requirements**

- **Operating System:** Linux (Ubuntu 20.04+, Debian 11+, CentOS 8+, RHEL 8+)
- **PHP:** 8.2 or higher
- **PostgreSQL:** 9.6 or higher (must have access to GreenArrow MTA database)
- **Web Server:** Apache 2.4+ or Nginx 1.18+
- **Composer:** Latest version

### **Required PHP Extensions**

- `pdo`
- `pdo_pgsql` (PostgreSQL driver)
- `mbstring`
- `openssl`
- `curl`
- `json`
- `intl` (recommended)

### **Additional Requirements**

- **GreenArrow MTA:** Active installation with PostgreSQL database
- **Database Access:** Read-only access to GreenArrow database
- **License Key:** Valid ArrowPulse license key
- **OpenAI API Key:** Optional, for AI-powered insights

---

## 🖥️ Server Preparation

### **Step 1: Update System Packages**

```bash
# Ubuntu/Debian
sudo apt update && sudo apt upgrade -y

# CentOS/RHEL
sudo yum update -y
```

### **Step 2: Install PHP 8.2+ and Required Extensions**

#### **Ubuntu/Debian:**

```bash
# Add PHP repository
sudo apt install -y software-properties-common
sudo add-apt-repository ppa:ondrej/php -y
sudo apt update

# Install PHP and extensions
sudo apt install -y php8.2 php8.2-cli php8.2-fpm php8.2-common \
    php8.2-pgsql php8.2-mbstring php8.2-xml php8.2-curl \
    php8.2-zip php8.2-gd php8.2-intl php8.2-bcmath

# Verify installation
php -v
php -m | grep -E 'pdo|pgsql|mbstring|openssl|curl|json|intl'
```

#### **CentOS/RHEL:**

```bash
# Install EPEL repository
sudo yum install -y epel-release

# Install Remi repository for PHP 8.2
sudo yum install -y https://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module reset php -y
sudo yum module enable php:remi-8.2 -y

# Install PHP and extensions
sudo yum install -y php php-cli php-fpm php-common php-pgsql \
    php-mbstring php-xml php-curl php-zip php-gd php-intl php-bcmath

# Verify installation
php -v
php -m | grep -E 'pdo|pgsql|mbstring|openssl|curl|json|intl'
```

### **Step 3: Install PostgreSQL Client**

```bash
# Ubuntu/Debian
sudo apt install -y postgresql-client

# CentOS/RHEL
sudo yum install -y postgresql
```

### **Step 4: Install Composer**

```bash
# Download and install Composer
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
sudo chmod +x /usr/local/bin/composer

# Verify installation
composer --version
```

### **Step 5: Install Web Server**

#### **Option A: Nginx (Recommended)**

```bash
# Ubuntu/Debian
sudo apt install -y nginx

# CentOS/RHEL
sudo yum install -y nginx

# Start and enable Nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```

#### **Option B: Apache**

```bash
# Ubuntu/Debian
sudo apt install -y apache2

# CentOS/RHEL
sudo yum install -y httpd

# Start and enable Apache
sudo systemctl start apache2  # or httpd on CentOS
sudo systemctl enable apache2 # or httpd on CentOS
```

### **Step 6: Create Application User (Recommended)**

```bash
# Create a dedicated user for ArrowPulse
sudo useradd -m -s /bin/bash arrowpulse

# Add user to web server group
sudo usermod -aG www-data arrowpulse  # Ubuntu/Debian
# OR
sudo usermod -aG nginx arrowpulse     # CentOS/RHEL (if using Nginx)
```

### **Step 7: Prepare Application Directory**

```bash
# Create application directory
sudo mkdir -p /var/www/arrowpulse
sudo chown arrowpulse:arrowpulse /var/www/arrowpulse

# Switch to application user
sudo su - arrowpulse
cd /var/www/arrowpulse
```

---

## 📦 Installation Steps

### **Step 1: Download ArrowPulse**

You can download ArrowPulse in two ways:

#### **Option A: Download Release Archive (Recommended for Production)**

```bash
# Download the latest release
cd /var/www/arrowpulse
wget https://github.com/DevHeaven/ArrowPulse/archive/refs/tags/v1.0.0.zip

# Extract the archive
unzip v1.0.0.zip
mv ArrowPulse-1.0.0 arrowpulse
cd arrowpulse

# Clean up
rm v1.0.0.zip
```

### **Step 2: Run Installation Script**

**This is the critical step!** The installation script prepares everything before the setup wizard.

```bash
# Make the script executable
chmod +x install.sh

# Run the installation script
./install.sh
```

**What the script does:**

1. ✅ Installs PHP dependencies via Composer
2. ✅ Creates `.env` file from template
3. ✅ Generates application encryption key
4. ✅ Creates required storage directories
5. ✅ Downloads license public key (if available)
6. ✅ Sets proper file permissions
7. ✅ Verifies PHP extensions

**Expected Output:**

```
🚀 ArrowPulse Automated Installation
═══════════════════════════════════════════════════════

✓ Composer found

📦 Step 1: Installing PHP dependencies...
✓ PHP dependencies installed

⚙️  Step 2: Checking environment configuration...
✓ Created .env file from .env.example

🔑 Step 3: Checking application key...
✓ Application key generated

📁 Step 4: Creating required directories...
✓ Directories created

🔐 Step 4b: Checking license public key...
✓ License public key downloaded successfully

🔐 Step 5: Setting file permissions...
✓ Permissions set

🔍 Step 6: Checking PHP extensions...
  ✓ PDO (available)
  ✓ pdo_pgsql
  ✓ mbstring
  ✓ openssl
  ✓ curl
  ✓ json

═══════════════════════════════════════════════════════
✅ Installation complete!
```

**If you see errors:** Refer to the [Troubleshooting](#troubleshooting) section below.

### **Step 3: Verify Installation**

```bash
# Check that .env file exists
ls -la .env

# Verify application key was generated
grep APP_KEY .env

# Check storage directories were created
ls -la storage/
ls -la bootstrap/cache/
```

---

## 🧙 Setup Wizard

After running `install.sh`, you're ready to access the setup wizard.

### **Step 1: Access Setup Wizard**

#### **Option A: Using PHP Built-in Server (Testing Only)**

```bash
php artisan serve --host=0.0.0.0 --port=8000
```

Then visit: `http://your-server-ip:8000/setup`

#### **Option B: Using Your Web Server (Production)**

Configure your web server first (see [Web Server Configuration](#web-server-configuration) below), then visit:

```
http://your-domain.com/setup
```

**Note:** The setup wizard will automatically redirect you if accessed directly.

### **Step 2: Step-by-Step Wizard Guide**

#### **Wizard Step 1: License Activation**

1. Enter your ArrowPulse license key
2. Click "Activate License"
3. Wait for confirmation

**Troubleshooting:**
- If activation fails, verify your license key
- Check internet connectivity
- Ensure license server is reachable: `https://licensing.arrowpulse.io`

#### **Wizard Step 2: System Requirements**

The wizard automatically checks:

- ✅ PHP version (8.2+)
- ✅ Required PHP extensions
- ✅ Directory permissions
- ✅ Application key

**If checks fail:** Fix the issues shown and refresh the page.

#### **Wizard Step 3: GreenArrow Database Connection**

1. **Host:** GreenArrow PostgreSQL server hostname or IP
   - Example: `localhost` or `192.168.1.100`
   
2. **Port:** PostgreSQL port (default: `5432`)

3. **Database:** GreenArrow database name
   - This is your existing GreenArrow MTA database
   - ArrowPulse will use **read-only** access

4. **Username:** PostgreSQL username with read access

5. **Password:** PostgreSQL password

6. Click **"Test Connection"** to verify

**Important Notes:**
- ArrowPulse requires read-only access to GreenArrow database
- Recommended: Create a dedicated PostgreSQL user with SELECT-only permissions
- The wizard verifies required GreenArrow tables exist

#### **Wizard Step 4: ArrowPulse Database Setup**

You have two options:

**Option A: Create New Database (Recommended)**

1. Enter PostgreSQL connection details (usually same as GreenArrow)
2. Enter desired database name (e.g., `arrowpulse`)
3. Click **"Create Database"**
4. The wizard will create the database and run migrations automatically

**Option B: Use Existing Database**

1. Enter PostgreSQL connection details
2. Enter existing database name
3. Click **"Test Connection"**
4. Click **"Save & Continue"** (migrations will run automatically)

**What happens:**
- Creates all required tables
- Sets up initial data structures
- Creates migration tracking

#### **Wizard Step 5: Administrator Account**

Create your first admin user:

1. **Name:** Your full name
2. **Email:** Admin email address (used for login)
3. **Password:** Strong password (minimum 8 characters)
4. **Confirm Password:** Re-enter password

**Security Tip:** Use a strong, unique password and store it securely.

#### **Wizard Step 6: Deployment Mode**

Choose your deployment type:

**Single Sender Mode:**
- For single organizations managing their own email
- All data is visible to all users
- Simpler configuration

**ESP Mode (Email Service Provider):**
- For managing multiple customers
- Automatic customer data isolation
- Customer selection interface

Select your mode and click **"Continue"**

#### **Wizard Step 7: Optional Features**

**AI Insights (Optional):**

1. Toggle **"Enable AI Insights"** if desired
2. Enter OpenAI API key (if enabling)
3. Select AI model (GPT-4o or GPT-4o Mini)
4. Click **"Test API Key"** to verify

**Note:** AI Insights can be configured later in Settings.

#### **Wizard Step 8: Complete Setup**

Review your configuration and click **"Complete Setup"**

The wizard will:
- ✅ Create setup completion lock file
- ✅ Redirect you to login page

---

## 🌐 Web Server Configuration

### **Nginx Configuration**

Create `/etc/nginx/sites-available/arrowpulse`:

```nginx
server {
    listen 80;
    server_name your-domain.com;
    root /var/www/arrowpulse/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_hide_header X-Powered-By;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

Enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/arrowpulse /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### **Apache Configuration**

Create `/etc/apache2/sites-available/arrowpulse.conf`:

```apache
<VirtualHost *:80>
    ServerName your-domain.com
    DocumentRoot /var/www/arrowpulse/public

    <Directory /var/www/arrowpulse/public>
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/arrowpulse-error.log
    CustomLog ${APACHE_LOG_DIR}/arrowpulse-access.log combined
</VirtualHost>
```

Enable the site:

```bash
sudo a2ensite arrowpulse.conf
sudo a2enmod rewrite
sudo apache2ctl configtest
sudo systemctl reload apache2
```

### **Set Proper Permissions**

```bash
# Set ownership
sudo chown -R arrowpulse:www-data /var/www/arrowpulse
sudo chown -R arrowpulse:nginx /var/www/arrowpulse  # If using Nginx

# Set directory permissions
sudo find /var/www/arrowpulse -type d -exec chmod 755 {} \;

# Set file permissions
sudo find /var/www/arrowpulse -type f -exec chmod 644 {} \;

# Make storage writable
sudo chmod -R 775 /var/www/arrowpulse/storage
sudo chmod -R 775 /var/www/arrowpulse/bootstrap/cache
```

### **SSL/HTTPS Setup (Production Required)**

```bash
# Install Certbot
sudo apt install -y certbot python3-certbot-nginx  # For Nginx
# OR
sudo apt install -y certbot python3-certbot-apache # For Apache

# Obtain certificate
sudo certbot --nginx -d your-domain.com  # For Nginx
# OR
sudo certbot --apache -d your-domain.com # For Apache

# Auto-renewal is configured automatically
```

---

## ⚙️ Queue Workers Setup

ArrowPulse uses background jobs for:
- AI analysis processing
- Engagement data aggregation
- Domain statistics processing

### **Option A: Laravel Horizon (Recommended)**

Horizon provides a web dashboard for monitoring queues.

```bash
# Install Supervisor (process manager)
sudo apt install -y supervisor

# Create Horizon configuration
sudo nano /etc/supervisor/conf.d/arrowpulse-horizon.conf
```

Add this configuration:

```ini
[program:arrowpulse-horizon]
process_name=%(program_name)s
command=php /var/www/arrowpulse/artisan horizon
autostart=true
autorestart=true
user=arrowpulse
redirect_stderr=true
stdout_logfile=/var/www/arrowpulse/storage/logs/horizon.log
stopwaitsecs=3600
```

Enable and start Supervisor:

```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start arrowpulse-horizon
```

**Access Horizon Dashboard:**

1. Ensure you're logged in as admin
2. Visit: `http://your-domain.com/horizon`
3. Monitor queue performance in real-time

### **Option B: Standard Queue Worker**

```bash
# Create queue worker configuration
sudo nano /etc/supervisor/conf.d/arrowpulse-worker.conf
```

Add this configuration:

```ini
[program:arrowpulse-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/arrowpulse/artisan queue:work --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=arrowpulse
numprocs=2
redirect_stderr=true
stdout_logfile=/var/www/arrowpulse/storage/logs/worker.log
stopwaitsecs=3600
```

Enable and start:

```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start arrowpulse-worker:*
```

---

## 🔄 Post-Installation

### **Step 1: Configure Laravel Scheduler**

ArrowPulse uses cron for scheduled tasks (engagement data processing, etc.)

```bash
# Edit crontab for arrowpulse user
sudo crontab -u arrowpulse -e

# Add this line (adjust path if needed)
* * * * * cd /var/www/arrowpulse && php artisan schedule:run >> /dev/null 2>&1
```

Verify it's working:

```bash
# Check cron is running
sudo systemctl status cron

# Test scheduler
cd /var/www/arrowpulse
php artisan schedule:run
```

### **Step 2: Configure Environment Variables**

Edit `.env` file for production:

```bash
cd /var/www/arrowpulse
nano .env
```

**Important Production Settings:**

```env
APP_ENV=production
APP_DEBUG=false
APP_URL=https://your-domain.com

LOG_CHANNEL=daily
LOG_LEVEL=error

# Database connections (already configured via wizard)
# ARROWPULSE_DB_* variables
# GREENARROW_DB_* variables

DB_CONNECTION=pgsql
```

### **Step 3: Optimize Application**

```bash
# Cache configuration
php artisan config:cache

# Cache routes
php artisan route:cache

# Cache views
php artisan view:cache

# Optimize autoloader
composer install --optimize-autoloader --no-dev
```

### **Step 4: Configure Additional Settings**

After logging in, configure:

1. **Settings → GreenArrow MTA**
   - Verify database connection
   - Test Stats API connection (if using Queue Inspector)

2. **Settings → SMTP** (Optional)
   - Configure email notifications
   - Test email delivery

3. **Settings → AI Insights** (Optional)
   - Enable/disable AI features
   - Configure OpenAI API key
   - Set model preferences

---

# Check Laravel logs
tail -f storage/logs/laravel.log

# Check Horizon logs (if using)
tail -f storage/logs/horizon.log

# Check system logs
tail -f /var/log/nginx/error.log  # For Nginx
# OR
tail -f /var/log/apache2/error.log # For Apache
```

---

## 🐛 Troubleshooting

### **Install Script Issues**

**Problem:** `install.sh` fails with "Composer not found"

```bash
# Solution: Install Composer first (see Server Preparation section)
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
```

**Problem:** Missing PHP extensions

```bash
# Check which extensions are missing
php -m

# Install missing extensions
sudo apt install -y php8.2-[extension-name]  # Ubuntu/Debian
sudo yum install -y php-[extension-name]     # CentOS/RHEL
```

**Problem:** Permission denied errors

```bash
# Fix ownership
sudo chown -R arrowpulse:www-data /var/www/arrowpulse

# Fix permissions
sudo chmod -R 775 storage bootstrap/cache
```

### **Setup Wizard Issues**

**Problem:** Cannot connect to GreenArrow database

- Verify PostgreSQL is running: `sudo systemctl status postgresql`
- Check firewall: `sudo ufw status` (Ubuntu) or `sudo firewall-cmd --list-all` (CentOS)
- Test connection manually: `psql -h hostname -U username -d database`
- Verify credentials in setup wizard
- Check PostgreSQL allows remote connections: `/etc/postgresql/*/main/pg_hba.conf`

**Problem:** License activation fails

- Verify license key is correct
- Check internet connectivity: `curl -I https://licensing.arrowpulse.io`
- Ensure license public key exists: `ls -la storage/app/private/license_public.pem`
- Check firewall allows outbound HTTPS

**Problem:** Database creation fails

- Verify PostgreSQL user has CREATE DATABASE permission
- Check database name doesn't already exist
- Verify PostgreSQL connection credentials

### **Web Server Issues**

**Problem:** 502 Bad Gateway

```bash
# Check PHP-FPM status
sudo systemctl status php8.2-fpm

# Check PHP-FPM configuration
sudo nano /etc/php/8.2/fpm/pool.d/www.conf
# Verify: listen = /var/run/php/php8.2-fpm.sock

# Restart PHP-FPM
sudo systemctl restart php8.2-fpm
```

**Problem:** 500 Internal Server Error

```bash
# Check Laravel logs
tail -50 storage/logs/laravel.log

# Clear caches
php artisan config:clear
php artisan cache:clear
php artisan view:clear

# Check file permissions
ls -la storage/ bootstrap/cache/
```

**Problem:** Permission denied errors

```bash
# Fix ownership
sudo chown -R arrowpulse:www-data /var/www/arrowpulse

# Fix permissions
sudo find /var/www/arrowpulse -type d -exec chmod 755 {} \;
sudo find /var/www/arrowpulse -type f -exec chmod 644 {} \;
sudo chmod -R 775 storage bootstrap/cache
```

### **Queue Worker Issues**

**Problem:** Jobs not processing

```bash
# Check Supervisor status
sudo supervisorctl status

# Check worker logs
tail -f storage/logs/horizon.log  # If using Horizon
tail -f storage/logs/worker.log   # If using standard worker

# Restart workers
sudo supervisorctl restart arrowpulse-horizon
# OR
sudo supervisorctl restart arrowpulse-worker:*
```

**Problem:** Horizon dashboard not accessible

- Verify you're logged in as admin user
- Check route: `php artisan route:list | grep horizon`
- Verify Horizon is running: `sudo supervisorctl status arrowpulse-horizon`

### **Database Connection Issues**

**Problem:** "Connection refused" errors

```bash
# Test PostgreSQL connectivity
psql -h hostname -U username -d database

# Check PostgreSQL is listening
sudo netstat -tlnp | grep 5432

# Verify connection settings in .env
grep -E "DB_|ARROWPULSE_DB_|GREENARROW_DB_" .env
```

### **Performance Issues**

**Problem:** Slow page loads

```bash
# Enable OPcache (if not already)
sudo nano /etc/php/8.2/fpm/php.ini
# Set: opcache.enable=1

# Increase PHP-FPM workers
sudo nano /etc/php/8.2/fpm/pool.d/www.conf
# Increase: pm.max_children = 50

# Clear caches
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

---

## 🔒 Security Checklist

Before going live, ensure:

- ✅ `APP_DEBUG=false` in `.env`
- ✅ `APP_ENV=production` in `.env`
- ✅ HTTPS/SSL certificate installed
- ✅ Strong admin password set
- ✅ Database credentials secure (not in version control)
- ✅ File permissions properly set (storage writable, code read-only)
- ✅ Firewall configured (only ports 80, 443, 22 open)
- ✅ Regular backups configured
- ✅ PostgreSQL read-only user for GreenArrow database
- ✅ License public key present and valid

---

## 📞 Support

If you encounter issues not covered in this guide:

1. Check `storage/logs/laravel.log` for detailed error messages
2. Review the [Troubleshooting](#troubleshooting) section above
3. Contact support: support@teneo.be

**When contacting support, please provide:**
- Error messages from logs
- Server environment details
- Steps to reproduce the issue
- Screenshots (if applicable)

---

## 📚 Additional Resources

- **GitHub Repository:** https://github.com/DevHeaven/arrowpulse_releases
- **Latest Release:** https://github.com/DevHeaven/arrowpulse_releases/releases/latest

---

**Congratulations! 🎉**

You've successfully installed ArrowPulse in production. The application is now ready to monitor your GreenArrow MTA and provide powerful insights into your email delivery performance.

**Next Steps:**
1. Log in with your admin credentials
2. Explore the dashboard
3. Configure additional settings as needed
4. Set up user accounts and permissions
5. Start monitoring your email infrastructure!

---

*Last Updated: November 2025*
*ArrowPulse Version: 1.0.0*

