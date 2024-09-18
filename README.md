# DockerWebserver

Let’s walk through setting up a local development environment with Docker that runs an Nginx server, PHP, and a MySQL database. We’ll be using **Docker Compose** to define and manage the different services. Here’s a step-by-step guide:

### 1. **Install Docker and Docker Compose**
Before we start, ensure you have Docker Desktop installed on your Windows 11 machine:
   - [Download Docker Desktop](https://www.docker.com/products/docker-desktop/)
   - Follow the installation instructions and make sure Docker is running properly after installation.

### 2. **Create a Project Directory**
You’ll need a directory to hold your Docker setup and project files. Open your terminal (Command Prompt, PowerShell, or Git Bash) and create a new directory for your project:

```bash
mkdir my_docker_project
cd my_docker_project
```

### 3. **Create `docker-compose.yml`**
This file will define your Nginx server, PHP service, and MySQL database. Inside your `my_docker_project` directory, create a file named `docker-compose.yml`.

Here’s a basic configuration for your setup:

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:latest
    container_name: nginx_container
    ports:
      - "8080:80"  # Exposes Nginx on port 8080 of your host machine
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf  # Nginx config file
      - ./app:/var/www/html  # Your PHP application directory
    depends_on:
      - php
    networks:
      - mynetwork

  php:
    image: php:8.1-fpm
    container_name: php_container
    volumes:
      - ./app:/var/www/html  # The same app folder is shared with PHP
    networks:
      - mynetwork

  mysql:
    image: mysql:8.0
    container_name: mysql_container
    environment:
      MYSQL_ROOT_PASSWORD: root_password
      MYSQL_DATABASE: my_database
      MYSQL_USER: user
      MYSQL_PASSWORD: user_password
    volumes:
      - mysql_data:/var/lib/mysql
    ports:
      - "3306:3306"  # Exposes MySQL on port 3306 of your host machine
    networks:
      - mynetwork

networks:
  mynetwork:
    driver: bridge

volumes:
  mysql_data:
```

### Explanation:
- **nginx**: Runs an Nginx container and serves PHP from the `php-fpm` service.
- **php**: Runs PHP-FPM (FastCGI Process Manager), which allows Nginx to serve PHP files.
- **mysql**: Runs a MySQL container with environment variables to set the root password, a database, and user credentials.
- **volumes**: These map directories on your host machine to directories inside the container. This allows you to work with files on your host machine and have them available to Nginx and PHP.

### 4. **Create Nginx Configuration**
You need to create an Nginx configuration file to tell Nginx how to handle PHP requests. Inside your project directory, create a folder called `nginx` and then a file named `nginx.conf`.

```bash
mkdir nginx
touch nginx/nginx.conf
```

Here’s a basic configuration to add to `nginx.conf`:

```nginx
worker_processes 1;

events { worker_connections 1024; }

http {
    server {
        listen 80;

        root /var/www/html;
        index index.php index.html;

        server_name localhost;

        location / {
            try_files $uri $uri/ =404;
        }

        location ~ \.php$ {
            include fastcgi_params;
            fastcgi_pass php:9000;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        }

        location ~ /\.ht {
            deny all;
        }
    }
}
```

### 5. **Create the PHP Application Directory**
Now, you’ll set up the PHP application directory. In your project folder, create an `app` directory and a simple PHP file to test:

```bash
mkdir app
touch app/index.php
```

Add the following content to the `index.php` file to ensure PHP is working:
(Please note that you should never expose phpinfo() on a live server, as it will output environment variables.)

```php
<?php
phpinfo();
?>
```

### 6. **Run Docker Compose**
Now that everything is set up, it’s time to start the containers using Docker Compose. From your project directory, run:

```bash
docker-compose up -d
```

This will:
- Pull the necessary images (`nginx`, `php-fpm`, and `mysql`).
- Create and start the containers.
- Run Nginx, PHP, and MySQL as separate services.

### 7. **Access the Web Server**
Once the containers are running, open your web browser and go to:

```
http://localhost:8080
```

You should see the **PHP info page**, which indicates that Nginx is correctly serving the PHP content.

### 8. **Connect to MySQL**
To connect to MySQL, you can use any MySQL client (e.g., MySQL Workbench, phpMyAdmin, or the command line) with the following credentials:

- **Host**: `localhost`
- **Port**: `3306`
- **User**: `user`
- **Password**: `user_password`
- **Database**: `my_database`

### 9. **Stop the Services**
When you’re done and want to stop the containers, simply run:

```bash
docker-compose down
```

This will stop and remove the containers, but the volumes (like the MySQL data) will persist.

### Summary of Steps:
1. Install Docker.
2. Create a project directory.
3. Define services in `docker-compose.yml`.
4. Create Nginx configuration (`nginx.conf`).
5. Create a basic PHP application (`index.php`).
6. Start the services with Docker Compose.
7. Test the web server on `localhost:8080`.
8. Connect to MySQL via any MySQL client.
9. Stop the services with `docker-compose down`.
