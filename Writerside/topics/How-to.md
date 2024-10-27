# How to Install XeroInput
Below is an updated step-by-step guide to installing a Laravel application on an Ubuntu system that includes both
Apache and Nginx setup options. This guide assumes PHP and MySQL are already installed.


## Before you start

To run this Laravel project, ensure your environment meets the following requirements:

- **PHP** version **8.2** or higher
- **Composer** for dependency management
- A supported **web server** like Apache or Nginx
- **MySQL** or **PostgreSQL** for the database

### Required PHP Extensions
Make sure these extensions are installed and enabled:
- `mbstring`
- `openssl`
- `pdo`
- `tokenizer`
- `xml`

For optional features, you may need:
- **Node.js** (for frontend assets if using `laravel/ui`)
- **Docker** (if using `laravel/sail` for local development)

### Additional Packages
This project uses various Laravel packages, including:
- **Laravel Sanctum** for API authentication
- **Laravel UI** for UI scaffolding
- **GuzzleHTTP** for HTTP requests
- **Maatwebsite Excel** for handling spreadsheets

Here's a condensed and structured guide for installing a Laravel application, complete with both Apache and Nginx server options and the cPanel setup, along with steps for importing databases.

---

## **Installation Steps for Laravel on Ubuntu**

### **Prerequisites**
Ensure PHP and MySQL are installed on your Ubuntu system.

### **Step 1: Update System Packages**
```bash
sudo apt update
sudo apt upgrade -y
```

### **Step 2: Install Composer**
```bash
sudo apt install curl -y
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
composer --version  # Verify installation
```

### **Step 3: Install Laravel**
```bash
cd /var/www/html
composer create-project --prefer-dist laravel/laravel xeroInput
```

### **Step 4: Set Directory Permissions**
```bash
sudo chown -R www-data:www-data /var/www/html/xeroInput
sudo chmod -R 755 /var/www/html/xeroInput/storage
sudo chmod -R 755 /var/www/html/xeroInput/bootstrap/cache
```

### **Step 5: Configure Environment**
1. Navigate to the Laravel project directory:
    ```bash
    cd /var/www/html/xeroInput
    cp .env.example .env
    php artisan key:generate
    ```
2. Update database configuration in `.env`:
    ```plaintext
    DB_CONNECTION=mysql
    DB_HOST=127.0.0.1
    DB_PORT=3306
    DB_DATABASE=xero_input
    DB_USERNAME=your_database_user
    DB_PASSWORD=your_database_password
    ```

### **Step 6: Create Database**
1. Access MySQL and create the database:
    ```bash
    sudo mysql -u root -p
    ```
2. Run the following in the MySQL shell:
    ```sql
    CREATE DATABASE xero_input;
    GRANT ALL PRIVILEGES ON xero_input.* TO 'your_database_user'@'localhost' IDENTIFIED BY 'your_database_password';
    FLUSH PRIVILEGES;
    EXIT;
    ```

### **Step 7: Configure Web Server (Choose Apache or Nginx)**

#### **Option 1: Apache**
1. Install and configure Apache:
    ```bash
    sudo apt install apache2 -y
    sudo a2enmod rewrite
    sudo nano /etc/apache2/sites-available/xeroInput.conf
    ```
2. Add this configuration in `xeroInput.conf`:
    ```apache
    <VirtualHost *:80>
        ServerAdmin admin@example.com
        DocumentRoot /var/www/html/xeroInput/public
        ServerName example.com
        ServerAlias www.example.com
        <Directory /var/www/html/xeroInput>
            Options Indexes FollowSymLinks
            AllowOverride All
            Require all granted
        </Directory>
    </VirtualHost>
    ```
3. Enable the new site and reload Apache:
    ```bash
    sudo a2ensite xeroInput.conf
    sudo systemctl reload apache2
    ```

#### **Option 2: Nginx**
1. Install Nginx and create a configuration file:
    ```bash
    sudo apt install nginx -y
    sudo nano /etc/nginx/sites-available/xeroInput
    ```
2. Add the following configuration in `xeroInput`:
    ```nginx
    server {
        listen 80;
        server_name example.com www.example.com;
        root /var/www/html/xeroInput/public;
        index index.php index.html index.htm;
        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }
        location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
        }
    }
    ```
3. Enable the site and restart Nginx:
    ```bash
    sudo ln -s /etc/nginx/sites-available/xeroInput /etc/nginx/sites-enabled/
    sudo nginx -t
    sudo systemctl restart nginx
    ```

---

## **Setting Up Laravel on cPanel**

1. **Log in to cPanel**: Access via `http://yourdomain.com/cpanel`.
2. **Create Database and User** in MySQL® Databases section.
3. **Install Composer**: Use cPanel Terminal to verify or install Composer:
    ```bash
    composer --version
    ```

4. **Upload Laravel Files**:
   - Use File Manager to upload and extract files to your subdomain directory.
   - Alternatively, use FTP to upload the Laravel files.

5. **Configure Environment Variables**:
   - Rename `.env.example` to `.env`.
   - Edit database configuration in `.env`.

6. **Set Permissions for Directories**:
   - Set 755 permissions on `storage` and `bootstrap/cache` directories.

7. **Update .htaccess**:
   - Ensure `.htaccess` contains:
      ```apache
      <IfModule mod_rewrite.c>
      RewriteEngine On
      RewriteCond %{REQUEST_FILENAME} !-d
      RewriteCond %{REQUEST_FILENAME} !-f
      RewriteRule ^ index.php [L]
      </IfModule>
      ```

8. **Set Up Cron Jobs**:
   ```bash
   * * * * * php /home/username/path_to_laravel_project/artisan schedule:run >> /dev/null 2>&1
   ```

---

## **Final Step: Database Configuration**

### **Option 1: Import SQL**
1. Transfer `xeroInput.sql` to the server and import in MySQL:
    ```bash
    sudo mysql -u root -p
    USE xero_input;
    SOURCE /path/to/xeroInput.sql;
    ```

### **Option 2: Run Migrations and Seeders**
1. Ensure `.env` is configured and run:
    ```bash
    php artisan migrate:fresh --seed
    ```

Access the application by navigating to your server's IP or domain.

Congratulations! You’ve successfully set up your Laravel application.