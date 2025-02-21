<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400" alt="Laravel Logo"></a></p>

<p align="center">
<a href="https://github.com/laravel/framework/actions"><img src="https://github.com/laravel/framework/workflows/tests/badge.svg" alt="Build Status"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/dt/laravel/framework" alt="Total Downloads"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/v/laravel/framework" alt="Latest Stable Version"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/l/laravel/framework" alt="License"></a>
</p>

# Laravel Development with Docker (WSL & Docker Desktop)

This guide provides step-by-step instructions for setting up **Laravel** in **WSL (Windows Subsystem for Linux)** and **Docker Desktop for Windows/Mac**.

---

## 📌 Prerequisites

Before starting, ensure you have:

- **WSL Installed (Ubuntu recommended)** → [Install WSL](https://learn.microsoft.com/en-us/windows/wsl/install)
- **Docker Installed**
  - [Install Docker for Windows/Mac](https://www.docker.com/get-started)
  - OR [Install Docker for WSL](https://docs.docker.com/desktop/wsl/)
- **Windows Terminal or VS Code with Remote WSL Extension**

---

## 🚀 **1. Clone the Laravel Project**
Inside **WSL (Ubuntu) or macOS/Linux terminal**, navigate to your projects directory and clone the Laravel repo:

```bash
git clone https://github.com/your-repo/laravel-app.git
cd laravel-app
```

If you’re setting up a new Laravel project, run:

```bash
composer create-project --prefer-dist laravel/laravel laravel-app
cd laravel-app
```

## 🐳 2. Setting Up Docker for Laravel

### ✅ Option 1: Using Docker in WSL
If you installed Docker in WSL, ensure the Docker service is running:

```bash
sudo service docker start
```

### ✅ Option 2: Using Docker Desktop (Windows/Mac)
Enable WSL Integration:

1. Open Docker Desktop
2. Go to Settings → Resources → WSL Integration
3. Enable Docker for your WSL distribution (Ubuntu)

Run Docker Commands in WSL

Open Windows Terminal (or VS Code Terminal) and switch to WSL:

```bash
wsl
```

Verify Docker is working:

```bash
docker --version
```

## 📌 3. Create Docker Configuration Files
Inside your Laravel project root, create the following files:

### 📄 Dockerfile

```dockerfile
# Use PHP 8.3 with FPM
FROM php:8.3-fpm

# Install dependencies
RUN apt-get update && apt-get install -y \
    curl zip unzip git \
    libpng-dev libjpeg-dev libfreetype6-dev libzip-dev \
    && docker-php-ext-install pdo_mysql mbstring zip exif pcntl gd

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Set working directory
WORKDIR /var/www

# Copy Laravel files
COPY . .

# Set permissions
RUN chown -R www-data:www-data storage bootstrap/cache

# Expose PHP-FPM port
EXPOSE 9000

CMD ["php-fpm"]
```

### 📄 docker-compose.yml

```yaml
version: '3.8'

services:
  app:
    build: .
    container_name: laravel_app
    restart: unless-stopped
    working_dir: /var/www
    volumes:
      - .:/var/www
    depends_on:
      - db
    networks:
      - laravel_network

  webserver:
    image: nginx:alpine
    container_name: laravel_nginx
    restart: unless-stopped
    ports:
      - "8000:80"
    volumes:
      - .:/var/www
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - app
    networks:
      - laravel_network

  db:
    image: mysql:8.0
    container_name: laravel_mysql
    restart: unless-stopped
    environment:
      MYSQL_DATABASE: laravel_db
      MYSQL_USER: laravel_user
      MYSQL_PASSWORD: secret
      MYSQL_ROOT_PASSWORD: root
    ports:
      - "3306:3306"
    networks:
      - laravel_network
    volumes:
      - db_data:/var/lib/mysql

networks:
  laravel_network:

volumes:
  db_data:
```

### 📄 nginx/default.conf

```nginx
server {
    listen 80;
    server_name localhost;

    root /var/www/public;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass app:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

## 🚀 4. Build & Start Docker Containers
Run the following command inside WSL or your terminal:

```bash
docker-compose up -d --build
```

This will:

- Start Laravel (PHP-FPM)
- Start Nginx (Web Server)
- Start MySQL Database

## ✅ 5. Install Laravel Dependencies
After Docker containers are up, install Laravel dependencies:

```bash
docker exec -it laravel_app composer install
docker exec -it laravel_app cp .env.example .env
docker exec -it laravel_app php artisan key:generate
```

## 🛠 6. Fix Permissions
If you get permission errors, run:

```bash
docker exec -it laravel_app chmod -R 775 storage bootstrap/cache
docker exec -it laravel_app chown -R www-data:www-data storage bootstrap/cache
```

Then restart Docker:

```bash
docker-compose restart
```

## ✅ 7. Run Database Migrations
Run migrations to create tables in MySQL:

```bash
docker exec -it laravel_app php artisan migrate --force
```

If MySQL connection issues occur, ensure `.env` has:

```env
DB_CONNECTION=mysql
DB_HOST=db   # Not localhost!
DB_PORT=3306
DB_DATABASE=laravel_db
DB_USERNAME=laravel_user
DB_PASSWORD=secret
```

Then clear cache:

```bash
docker exec -it laravel_app php artisan config:clear
```

## 🎯 8. Access Laravel in Your Browser
Now, open:

```arduino
http://localhost:8000
```

If everything is set up correctly, your Laravel app should load! 🚀

## 📌 9. Managing Docker Containers

| Command                                    | Description                        |
|--------------------------------------------|------------------------------------|
| `docker-compose up -d`                     | Start all containers in the background |
| `docker-compose down`                      | Stop and remove all containers     |
| `docker-compose restart`                   | Restart all containers             |
| `docker exec -it laravel_app bash`         | Access Laravel container shell     |
| `docker exec -it laravel_mysql mysql -u root -p` | Access MySQL shell                 |
| `docker logs laravel_app`                  | View Laravel logs                  |

## 🎉 Congratulations! Your Laravel App is Running in Docker
Your Laravel application is now containerized, making it easy to deploy and portable across Windows, Mac, and Linux!

If you encounter any issues, check logs:

```bash
docker logs laravel_app
docker logs laravel_mysql
docker logs laravel_nginx
```

🚀 Happy coding! Let me know if you need any modifications! 😊🔥

---

### **🔹 What’s New in This README?**
✔️ **Guidance for WSL & Docker Desktop**  
✔️ **Clear steps for Laravel setup**  
✔️ **MySQL connection fixes**  
✔️ **Common troubleshooting & logs**  

This will make it **easy for anyone** to set up Laravel on **WSL, Docker Desktop (Windows/Mac), or Linux**. Let me know if you want any tweaks! 🚀🔥

## About Laravel

Laravel is a web application framework with expressive, elegant syntax. We believe development must be an enjoyable and creative experience to be truly fulfilling. Laravel takes the pain out of development by easing common tasks used in many web projects, such as:

- [Simple, fast routing engine](https://laravel.com/docs/routing).
- [Powerful dependency injection container](https://laravel.com/docs/container).
- Multiple back-ends for [session](https://laravel.com/docs/session) and [cache](https://laravel.com/docs/cache) storage.
- Expressive, intuitive [database ORM](https://laravel.com/docs/eloquent).
- Database agnostic [schema migrations](https://laravel.com/docs/migrations).
- [Robust background job processing](https://laravel.com/docs/queues).
- [Real-time event broadcasting](https://laravel.com/docs/broadcasting).

Laravel is accessible, powerful, and provides tools required for large, robust applications.

## Learning Laravel

Laravel has the most extensive and thorough [documentation](https://laravel.com/docs) and video tutorial library of all modern web application frameworks, making it a breeze to get started with the framework.

You may also try the [Laravel Bootcamp](https://bootcamp.laravel.com), where you will be guided through building a modern Laravel application from scratch.

If you don't feel like reading, [Laracasts](https://laracasts.com) can help. Laracasts contains thousands of video tutorials on a range of topics including Laravel, modern PHP, unit testing, and JavaScript. Boost your skills by digging into our comprehensive video library.

## Laravel Sponsors

We would like to extend our thanks to the following sponsors for funding Laravel development. If you are interested in becoming a sponsor, please visit the [Laravel Partners program](https://partners.laravel.com).

### Premium Partners

- **[Vehikl](https://vehikl.com/)**
- **[Tighten Co.](https://tighten.co)**
- **[WebReinvent](https://webreinvent.com/)**
- **[Kirschbaum Development Group](https://kirschbaumdevelopment.com)**
- **[64 Robots](https://64robots.com)**
- **[Curotec](https://www.curotec.com/services/technologies/laravel/)**
- **[Cyber-Duck](https://cyber-duck.co.uk)**
- **[DevSquad](https://devsquad.com/hire-laravel-developers)**
- **[Jump24](https://jump24.co.uk)**
- **[Redberry](https://redberry.international/laravel/)**
- **[Active Logic](https://activelogic.com)**
- **[byte5](https://byte5.de)**
- **[OP.GG](https://op.gg)**

## Contributing

Thank you for considering contributing to the Laravel framework! The contribution guide can be found in the [Laravel documentation](https://laravel.com/docs/contributions).

## Code of Conduct

In order to ensure that the Laravel community is welcoming to all, please review and abide by the [Code of Conduct](https://laravel.com/docs/contributions#code-of-conduct).

## Security Vulnerabilities

If you discover a security vulnerability within Laravel, please send an e-mail to Taylor Otwell via [taylor@laravel.com](mailto:taylor@laravel.com). All security vulnerabilities will be promptly addressed.

## License

The Laravel framework is open-sourced software licensed under the [MIT license](https://opensource.org/licenses/MIT).
