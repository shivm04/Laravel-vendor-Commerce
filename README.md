# Laravel Ecommerce Application Deployment On Amazon EC2 Instance

## Step 1:

This guide provides step-by-step instructions to deploy a Laravel e-commerce application on an Amazon EC2 instance.

---

## **Prerequisites**

1. **Amazon EC2 Instance**:
    - Ubuntu Server (20.04 or later recommended).
    - Security group configured with inbound rules for HTTP (port 80), HTTPS (port 443), 8000 (php port 8000)and SSH (port 22).
2. **Domain Name** (optional):
    - Configure your domain's DNS to point to the EC2 public IP.
3. **Software Installed on EC2**:
    - PHP (8.1 or later)
    - Composer
    - Apache
    - MySQL

---

## **Step 1: Update and Prepare the Server**

1. **Update the package list:**
    
    ```
    sudo apt update && sudo apt upgrade -y
    ```
    
2. **Install essential tools:**
    
    ```
    sudo apt install -y unzip curl git
    ```
    

---

## **Step 2: Install PHP and Required Extensions**

1. **Add PHP repository and install PHP:**
    
    ```
    sudo apt install -y software-properties-common
    sudo add-apt-repository ppa:ondrej/php -y
    sudo apt update
    sudo apt install php8.1 php8.1-cli php8.1-fpm php8.1-mbstring php8.1-xml php8.1-curl php8.1-zip php8.1-bcmath php8.1-intl php8.1-mysql php8.1-soap php8.1-gd -y
    ```
    
2. **Verify PHP installation:**
    
    ```
    php -v
    ```
    
3. **Install Supervisor:**
    
    ```
    curl -sS https://getcomposer.org/installer | php
    sudo mv composer.phar /usr/local/bin/composer
    composer -v
    
    composer global require laravel/installer
    ```
    
4. **Install Node JS and NPM:**
    
    ```
    sudo apt install nodejs npm -y
    ```
    

## **Step 3: Install MySQL**

1. **Install MySQL Server:**
    
    ```
    sudo apt install -y mysql-server
    ```
    
2. **Secure MySQL installation and root password setup:**
    
    ```
    sudo mysql_secure_installation
    sudo mysql
    ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'laravel@123';
    FLUSH PRIVILEGES;
    ```
    
3. **Create a database for Laravel:**
    
    ```
    sudo mysql -u root -p
    CREATE DATABASE laravel_ecommerce;
    CREATE USER 'laravel_user'@'%' IDENTIFIED BY 'secure_password';
    GRANT ALL PRIVILEGES ON laravel_ecommerce.* TO 'laravel_user'@'%';
    FLUSH PRIVILEGES;
    EXIT;
    ```
    

---

## **Step 4: Install Apache or Nginx**

1. **Install Apache:**
    
    ```
    sudo apt install -y apache2
    ```
    
2. **Enable required modules:**
    
    ```
    sudo a2enmod rewrite
    sudo a2enmod proxy
    sudo a2enmod proxy_fcgi
    sudo systemctl restart apache2
    ```
    
3. **Verify Apache configuration:**
    
    ```
    sudo apache2ctl -t
    sudo systemctl enable apache2
    sudo systemctl restart apache2
    ```
    

---

## **Step 5: Deploy Laravel Application**

1. **Clone your Laravel application from Git:**
    
    ```
    cd /var/www/
    sudo mkdir Laravel-vendor-Commerce
    sudo chown -R ubuntu:ubuntu Laravel-vendor-Commerce
    sudo git clone https://github.com/shivm04/Laravel-vendor-Commerce.git Laravel-vendor-Commerce
    ```
    
2. **Set correct permissions:**
    
    ```
    sudo chown -R ubuntu:ubuntu/var/www/Laravel-vendor-Commerce
    sudo chmod -R www-data:ubuntu /var/www/Laravel-vendor-Commerce/storage /var/www/Laravel-vendor-Commerce/bootstrap/cache
    sudo chmod -R 775 /var/www/Laravel-vendor-Commerce/storage /var/www/Laravel-vendor-Commerce/bootstrap/cache
    ```
    
3. **Install dependencies and Import database:**
    
    ```
    cd Sql\ File/
    cat eCommerce.sql | mysql -u root -p laravel_ecommerce
    composer install
    ```
    
4. **Configure environment file:**
    
    ```
    cp .env.example .env
    vi .env
    ```
    
    Update the database credentials and other necessary configurations in `.env`.
    
5. **Generate the application key:**
    
    ```
    php artisan key:generate
    ```
    
6. **Test the application:**
    
    ```
    php artisan serve --host=0.0.0.0 --port=8000
    ```
    

---

## **Step 6: Configure Apache for Laravel**

1. **Create a new Apache configuration file:**
    
    ```
    sudo vi /etc/apache2/sites-available/Laravel-vendor-Commerce.conf
    ```
    
    Add the following content:
    
    ```
    <VirtualHost *:80>
        ServerName laravel.cloudpita.com    
        DocumentRoot /var/www/Laravel-vendor-Commerce/public
    
        <Directory /var/www/Laravel-vendor-Commerce/public>
            Options +FollowSymlinks
            AllowOverride All
            Require all granted
                    <FilesMatch \.php$>
                    SetHandler "proxy:unix:/run/php/php8.1-fpm.sock|fcgi://localhost"
                    SetEnvIf Authorization "(.*)" HTTP_AUTHORIZATION=$1
                    SetEnvIf Content-Type "(.*)" HTTP_CONTENT_TYPE=$1
                    SetEnvIf Accept "(.*)" HTTP_ACCEPT=$1
            </FilesMatch>
        </Directory>
    
        ErrorLog ${APACHE_LOG_DIR}/Laravel-vendor-Commerce-error.log
        CustomLog ${APACHE_LOG_DIR}/Laravel-vendor-Commerce-access.log combined
    </VirtualHost>
    ```
    
2. **Enable the site and restart Apache:**
    
    ```
    sudo a2ensite Laravel-vendor-Commerce
    sudo systemctl reload apache2
    ```
    

---

## **Step 7: Configure Domain Name (Optional)**

Update your domain's DNS to point to the EC2 public IP. You can manage this through your domain registrar's control panel.

---

## **Step 8: Install SSL Certificate (Optional)**

1. **Install Certbot:**
    
    ```
    sudo apt install -y certbot python3-certbot-apache
    ```
    
2. **Obtain and configure SSL certificate:**
    
    ```
    sudo certbot --apache -d laravel.cloudpita.com
    ```
    

---

## **Step 9: Verify the Deployment**

1. **Access the application via the public IP or domain name:**
    
    ```
    http://laravel.cloudpita.com
    https://laravel.cloudpita.com
    
    Admin Username :- hamzawy1
    Admin Password :- admin1234
    ```
    
2. **Check Laravel logs and apache logs for issues:**
    
    ```
    tail -f /var/www/Laravel-vendor-Commerce/storage/logs/laravel.log
    tail -f /var/log/apache2/Laravel-vendor-Commerce-access.log
    ```
    

---
